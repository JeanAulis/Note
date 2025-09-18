```java
package com.zzyl.job;

import cn.hutool.core.collection.CollectionUtil;
import cn.hutool.core.util.NumberUtil;
import cn.hutool.core.util.ReflectUtil;
import cn.hutool.core.util.StrUtil;
import com.alibaba.fastjson.JSON;
import com.zzyl.dto.AlertRuleDto;
import com.zzyl.mapper.AlertRuleMapper;
import com.zzyl.mapper.UserMapper;
import com.zzyl.utils.ObjectUtil;
import com.zzyl.vo.DeviceDataVo;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;

import java.lang.reflect.Field;
import java.math.BigDecimal;
import java.time.LocalTime;
import java.util.ArrayList;
import java.util.List;
import java.util.Map;
import java.util.Set;
import java.util.stream.Collectors;

/**
 * @Description 报警任务，每分钟执行一次，根据规则对设备数据进行判断并生成报警记录
 * @Author jeanAulis
 * @Date 2025-09-16
 */
@Slf4j
@Component
public class AlarmsTask {
    @Autowired
    private AlertRuleMapper alertRuleMapper;

    @Autowired
    private RedisTemplate<String, String> redisTemplate;

    @Autowired
    private UserMapper userMapper; // 用于查询用户信息

    @Autowired
    private AlertRecordMapper alertRecordMapper; // 用于插入告警记录

    // 告警通知相关依赖
    @Autowired
    private NotificationService notificationService; // 通知服务

    // Redis key前缀常量
    private static final String REDIS_DEVICE_DATA_PREFIX = "device:data:";
    private static final String REDIS_ALERT_COUNT_PREFIX = "alert:count:";
    private static final String REDIS_SILENCE_PERIOD_PREFIX = "alert:silence:";
    private static final String DEVICE_UPLOAD_TIME_PREFIX = "device:upload:time:";

    /**
     * 每30秒执行一次，处理报警逻辑
     */
    @Scheduled(cron = "*/30 * * * * ?")
    public void alarmsTask() {
        try {
            // 1. 获取所有启用的报警规则,包含AlertRuleDto信息
            List<AlertRuleDto> enableRules = alertRuleMapper.getByEnableStatus();
            if (CollectionUtil.isEmpty(enableRules)) {
                log.info("未找到启用的报警规则");
                return;
            }

            // 2. 数据处理
            // 2.1 将规则按iotId分组
            Map<String, List<AlertRuleDto>> rulesByIotId = enableRules.stream()
                    .filter(r -> ObjectUtil.isNotEmpty(r.getIotId()))
                    .collect(Collectors.groupingBy(AlertRuleDto::getIotId));

            // 获取全局规则（iotId为空的规则）
            List<AlertRuleDto> globalRules = enableRules.stream()
                    .filter(r -> ObjectUtil.isEmpty(r.getIotId()))
                    .collect(Collectors.toList());

            Set<String> allIotIds = rulesByIotId.keySet();
            log.info("需要检查的设备数量：{}，设备ID列表：{}", allIotIds.size(), allIotIds);

            // 2.2 从Redis查询所有设备上传的数据，并进行数据格式标准化处理
            List<DeviceDataVo> allDeviceData = queryAndStandardizeDeviceData(allIotIds);
            if (CollectionUtil.isEmpty(allDeviceData)) {
                log.info("未查询到有效的设备数据");
                return;
            }

            // 3. 数据报警处理器
            for (DeviceDataVo deviceData : allDeviceData) {
                // 获取该设备的规则（设备特定规则 + 全局规则）
                List<AlertRuleDto> deviceRules = new ArrayList<>();
                if (rulesByIotId.containsKey(deviceData.getIotId())) {
                    deviceRules.addAll(rulesByIotId.get(deviceData.getIotId()));
                }
                deviceRules.addAll(globalRules);

                if (CollectionUtil.isEmpty(deviceRules)) {
                    log.debug("设备 {} 没有配置报警规则", deviceData.getIotId());
                    continue;
                }

                // 处理每个规则
                for (AlertRuleDto rule : deviceRules) {
                    processAlertRule(deviceData, rule);
                }
            }
        } catch (Exception e) {
            log.error("告警任务执行异常", e);
        }
    }

    /**
     * 查询并标准化设备数据
     */
    private List<DeviceDataVo> queryAndStandardizeDeviceData(Set<String> iotIds) {
        List<DeviceDataVo> deviceDataList = new ArrayList<>();
        long currentTime = System.currentTimeMillis();

        for (String iotId : iotIds) {
            try {
                // 从Redis获取设备数据
                String deviceDataJson = redisTemplate.opsForValue().get(REDIS_DEVICE_DATA_PREFIX + iotId);
                if (StrUtil.isEmpty(deviceDataJson)) {
                    continue;
                }

                // 解析设备数据
                DeviceDataVo deviceData = JSON.parseObject(deviceDataJson, DeviceDataVo.class);

                // 获取设备上传时间
                String uploadTimeStr = redisTemplate.opsForValue().get(DEVICE_UPLOAD_TIME_PREFIX + iotId);
                long uploadTime = NumberUtil.parseLong(uploadTimeStr, 0L);

                // 判断设备上传时间是否大于1分钟
                if (currentTime - uploadTime > 60000) {
                    log.debug("设备 {} 数据已超过1分钟未更新，跳过", iotId);
                    continue;
                }

                deviceData.setUploadTime(uploadTime);
                deviceDataList.add(deviceData);

            } catch (Exception e) {
                log.error("处理设备 {} 数据异常", iotId, e);
            }
        }

        return deviceDataList;
    }

    /**
     * 处理单个报警规则
     */
    private void processAlertRule(DeviceDataVo deviceData, AlertRuleDto rule) {
        try {
            // 1. 判断是否在生效时间
            if (!isInEffectivePeriod(rule.getAlertEffectivePeriod())) {
                log.debug("规则 {} 不在生效时间内", rule.getRuleName());
                return;
            }

            // 2. 判断数据是否达到阈值
            if (!checkThreshold(deviceData, rule)) {
                // 未达到阈值，删除Redis警报计数
                String countKey = buildAlertCountKey(deviceData.getIotId(), rule.getId());
                redisTemplate.delete(countKey);
                return;
            }

            // 3. 判断是否在沉默周期
            String silenceKey = buildSilencePeriodKey(deviceData.getIotId(), rule.getId());
            if (redisTemplate.hasKey(silenceKey)) {
                log.debug("规则 {} 还在沉默期内", rule.getRuleName());
                return;
            }

            // 4. 更新报警计数
            String countKey = buildAlertCountKey(deviceData.getIotId(), rule.getId());
            Long alertCount = redisTemplate.opsForValue().increment(countKey);

            // 设置过期时间（防止计数一直累积）
            redisTemplate.expire(countKey, 10, TimeUnit.MINUTES);

            // 5. 判断是否达到报警周期
            if (alertCount < rule.getAlertCycle()) {
                log.debug("规则 {} 当前报警次数 {}，未达到报警周期 {}",
                        rule.getRuleName(), alertCount, rule.getAlertCycle());
                return;
            }

            // 6. 触发报警
            log.info("触发报警：设备 {}，规则 {}", deviceData.getIotId(), rule.getRuleName());

            // 删除报警计数
            redisTemplate.delete(countKey);

            // 设置沉默期
            if (rule.getSilencePeriod() != null && rule.getSilencePeriod() > 0) {
                redisTemplate.opsForValue().set(silenceKey, "1",
                        rule.getSilencePeriod(), TimeUnit.MINUTES);
            }

            // 7. 发送通知并记录
            sendAlertNotification(deviceData, rule);

        } catch (Exception e) {
            log.error("处理报警规则异常，设备：{}，规则：{}",
                    deviceData.getIotId(), rule.getRuleName(), e);
        }
    }

    /**
     * 检查是否在生效时间内
     */
    private boolean isInEffectivePeriod(String effectivePeriod) {
        if (StrUtil.isEmpty(effectivePeriod)) {
            // 默认全天生效
            return true;
        }

        try {
            // 解析时间段格式：00:00:00~23:59:59
            String[] periods = effectivePeriod.split("~");
            if (periods.length != 2) {
                return true;
            }

            LocalTime startTime = LocalTime.parse(periods[0].trim());
            LocalTime endTime = LocalTime.parse(periods[1].trim());
            LocalTime currentTime = LocalTime.now();

            // 处理跨天的情况
            if (startTime.isAfter(endTime)) {
                return currentTime.isAfter(startTime) || currentTime.isBefore(endTime);
            } else {
                return !currentTime.isBefore(startTime) && !currentTime.isAfter(endTime);
            }
        } catch (Exception e) {
            log.error("解析生效时间异常：{}", effectivePeriod, e);
            return true;
        }
    }

    /**
     * 检查数据是否达到阈值
     */
    private boolean checkThreshold(DeviceDataVo deviceData, AlertRuleDto rule) {
        try {
            // 获取监控字段的值
            String fieldName = rule.getMonitorField();
            Object fieldValue = getFieldValue(deviceData, fieldName);

            if (fieldValue == null) {
                log.debug("设备 {} 未找到监控字段 {}", deviceData.getIotId(), fieldName);
                return false;
            }

            // 转换为数值进行比较
            BigDecimal value = new BigDecimal(fieldValue.toString());
            BigDecimal threshold = new BigDecimal(rule.getThresholdValue());

            // 根据比较操作符判断
            String operator = rule.getComparisonOperator();
            switch (operator) {
                case ">":
                    return value.compareTo(threshold) > 0;
                case ">=":
                    return value.compareTo(threshold) >= 0;
                case "<":
                    return value.compareTo(threshold) < 0;
                case "<=":
                    return value.compareTo(threshold) <= 0;
                case "=":
                case "==":
                    return value.compareTo(threshold) == 0;
                case "!=":
                case "<>":
                    return value.compareTo(threshold) != 0;
                default:
                    log.warn("未知的比较操作符：{}", operator);
                    return false;
            }
        } catch (Exception e) {
            log.error("检查阈值异常", e);
            return false;
        }
    }

    /**
     * 通过反射获取字段值
     */
    private Object getFieldValue(DeviceDataVo deviceData, String fieldName) {
        try {
            // 首先尝试从扩展数据中获取
            if (deviceData.getExtData() != null) {
                Object value = deviceData.getExtData().get(fieldName);
                if (value != null) {
                    return value;
                }
            }

            // 使用反射获取字段值
            Field field = ReflectUtil.getField(deviceData.getClass(), fieldName);
            if (field != null) {
                field.setAccessible(true);
                return field.get(deviceData);
            }

            return null;
        } catch (Exception e) {
            log.error("获取字段值异常：{}", fieldName, e);
            return null;
        }
    }

    /**
     * TODO发送告警通知
     */
    private void sendAlertNotification(DeviceDataVo deviceData, AlertRuleDto rule) {
        try {
            List<Long> notifyUserIds = new ArrayList<>();
            String alertType = rule.getAlertType();

            // 1. 获取超级管理员
            List<Long> adminIds = userMapper.getSuperAdminIds();
            notifyUserIds.addAll(adminIds);

            // 2. 根据告警类型确定通知人员
            if ("老人设备异常".equals(alertType)) {
                // 判断是否是固定设备
                if ("固定设备异常".equals(rule.getSubType())) {
                    // 通过床位查询老人护理员
                    List<Long> nurseIds = userMapper.getNurseIdsByBed(deviceData.getBedId());
                    notifyUserIds.addAll(nurseIds);
                } else {
                    // 通过老人查询护理员
                    List<Long> nurseIds = userMapper.getNurseIdsByElderly(deviceData.getElderlyId());
                    notifyUserIds.addAll(nurseIds);
                }
            } else {
                // 其他类型，通知维修员
                List<Long> repairmanIds = userMapper.getRepairmanIds();
                notifyUserIds.addAll(repairmanIds);
            }

            // 3. 发送通知
            if (CollectionUtil.isNotEmpty(notifyUserIds)) {
                // 构建通知内容
                String content = buildAlertContent(deviceData, rule);

                // 批量发送通知
                notificationService.sendBatchNotification(notifyUserIds, rule.getRuleName(), content);

                // 4. 批量插入告警记录
                List<AlertRecord> records = new ArrayList<>();
                for (Long userId : notifyUserIds) {
                    AlertRecord record = new AlertRecord();
                    record.setRuleId(rule.getId());
                    record.setRuleName(rule.getRuleName());
                    record.setIotId(deviceData.getIotId());
                    record.setDeviceName(deviceData.getDeviceName());
                    record.setAlertContent(content);
                    record.setAlertTime(new Date());
                    record.setUserId(userId);
                    record.setStatus(0); // 未处理
                    record.setAlertLevel(rule.getAlertLevel());
                    records.add(record);
                }

                if (CollectionUtil.isNotEmpty(records)) {
                    alertRecordMapper.insertBatch(records);
                }
            }

        } catch (Exception e) {
            log.error("发送告警通知异常", e);
        }
    }

    /**
     * 构建告警内容
     */
    private String buildAlertContent(DeviceDataVo deviceData, AlertRuleDto rule) {
        return String.format("设备【%s】触发告警规则【%s】，监控字段【%s】当前值【%s】%s阈值【%s】",
                deviceData.getDeviceName(),
                rule.getRuleName(),
                rule.getMonitorField(),
                getFieldValue(deviceData, rule.getMonitorField()),
                rule.getComparisonOperator(),
                rule.getThresholdValue());
    }

    /**
     * 构建告警计数Redis key
     */
    private String buildAlertCountKey(String iotId, Long ruleId) {
        return REDIS_ALERT_COUNT_PREFIX + iotId + ":" + ruleId;
    }

    /**
     * 构建沉默期Redis key
     */
    private String buildSilencePeriodKey(String iotId, Long ruleId) {
        return REDIS_SILENCE_PERIOD_PREFIX + iotId + ":" + ruleId;
    }

}
```



```java
package com.zzyl.job;

import com.zzyl.dto.AlertRuleDto;
import com.zzyl.mapper.AlertRuleMapper;
import com.zzyl.utils.ObjectUtil;
import com.zzyl.vo.DeviceDataVo;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;

import java.util.List;
import java.util.Map;
import java.util.Set;
import java.util.stream.Collectors;

/**
 * @Description 报警任务，每分钟执行一次，根据规则对设备数据进行判断并生成报警记录
 * @Author jeanAulis
 * @Date 2025-09-16
 */
@Slf4j
@Component
public class AlarmsTask {
    @Autowired
    private AlertRuleMapper alertRuleMapper;

    @Autowired
    private RedisTemplate<String, String> redisTemplate;

    // 告警通知相关依赖后续开启

    /**
     * 每分钟秒执行一次，处理报警逻辑
     */
    @Scheduled(cron = "*/30 * * * * ?")
    public void alarmsTask() {
        // 1. 获取所有启用的报警规则,包含AlertRuleDto信息
        List<AlertRuleDto> EnableRules = alertRuleMapper.getByEnableStatus();
        if (EnableRules == null || EnableRules.isEmpty()) {
            log.info("未找到启用的报警规则");
            return;
        }

        // 2. 数据处理
        // 2.1 将规则按iotId分组
        Map<String, List<AlertRuleDto>> rulesByIotId = EnableRules.stream()
                .filter(r -> ObjectUtil.isNotEmpty(r.getIotId()))
                .collect(Collectors.groupingBy(AlertRuleDto::getIotId));
        Set<String> allIotIds = rulesByIotId.keySet();
        log.info("allIotIds:{}", allIotIds);

        /**
         * 2.2 从Redis查询所有设备上传的数据，并进行数据格式标准化处理
         * 标准化处理具体是：
         * 数据转换为List<DeviceDataVo>,
         * 对每条数据进行校验，判断设备上传时间是否大于1分钟，大于则结束程序
         * 小于则将全部设备的报警规则和本设备的报警规则合并，合并规则判空则结束程序
         * 未判空则进入下一步的数据报警器
         */
        // TODO

        /**
         *  3. 数据报警处理器
         *  先判断是否在生效时间（alertEffectivePeriod:00:00:00~23:59:59）
         *  在判断数据是否达到阈值，未达到则删除Redis警报数，并结束
         *  达到之后判断是否在沉默周期（Redis查询）如果存在，则表示还没到可以报警的时间，并结束
         *  如果不存在报警时间，则查询当前报警数，计算当前报警数（当前报警数+=1）
         *  在判断报警数是否等于报警周期，不等则Redis报警数+1，并结束程序
         *  若相等，也就是全部校验通过，出发报警通知，
         *  删除Redis报警数，Redis添加沉默周期
         *  判断是"老人设备异常"吗？是的话判断是否是"固定设备异常"是的话通过床位查询老人护理员，不是固定设备异常的话通过老人来查询护理员，报警给护理员和超级管理员（先查询超级管理员），然后批量插入报警数据
         *  如果不是老人设备异常，则查询维修员，通知维修员和超级管理员（也要先获取超级管理员），批量插入报警数据
         */
        // TODO
    }

}
```