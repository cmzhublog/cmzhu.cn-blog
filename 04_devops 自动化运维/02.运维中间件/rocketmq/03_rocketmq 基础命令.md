## Rocketmq 基础命令

### rocketmq 命令行创建延迟队列

```bash
$	delaytipics="""delay_check_disburse_status_v1_topic
delay_disburse_notify_v1_topic
delay_repay_notify_v1_topic
delay_credit_apply_v1_topic
delay_credit_risk_apply_v1_topic
delay_appsflyer_callback_v1_topic
delay_notify_push_v1_topic
delay_notify_sms_v1_topic
delay_notify_sms_status_v1_topic"""



$ for i in $delaytipics ;do         echo "create delay topic of "$i;         /opt/rocketmq/ops_mqadmin.sh updateTopic -t $i -c DefaultCluster -n localhost:9876 -a +message.type=DELAY; done
```

