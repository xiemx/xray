---
title: Commands Tips
layout: post
comments: true
---


```
#获取所有region的机器数量
for i in $(aws ec2 describe-regions | grep RegionName| awk -F"[\"]+" '{print $4}');do echo -n "$i:  "; AWS_PROFILE=jp aws --region=$i ec2 describe-instances|grep InstanceId|wc -l;done;

#获取节点的机器tag.name和privateIP
aws ec2 describe-instances --filters "Name=tag:Service,Values=eos" --query "Reservations[*].Instances[*].[PrivateIpAddress,Tags[?Key=='Name'].Value]" --output text

#simple http server
python -m SimperHTTPServer 60123

#ssh端口转发
ssh -v -N -L 0.0.0.0:60123:192.168.10.195:60123 203.175.165.66

#查看rsa私钥的公钥
ssh-keygen -y -f id_rsa > id_rsa.pub

#wget 断点续传
wget -c -t 0 -O backup_2018_01_01_050000_9591273.bak http://47.88.192.116:12312/backup_2018_01_01_050000_9591273.bak


#rsync+ssh proxy
rsync -av -P -e "ssh -o 'ProxyCommand ssh 123.206.197.235 nc %h %p'" 2018-03-06_03-05.sql.gz 10.86.48.8:/data/backup

#consul-template
/srv/consul/consul-template -consul 192.168.10.123:8500 -template /srv/consul/my.ctmpl:/srv/consul/conf.d/my.conf:/etc/init.d/nginx reload -once

#iptables forward
iptables -t nat -A PREROUTING -d 192.168.10.226 -p tcp -dport 6386 -j DNAT --to-destination 192.168.10.81:6386
iptables -t nat -I POSTROUTING  -j MASQUERADE

#远程抓包在本地调用wireshark工具打开
ssh -o 'ProxyCommand ssh 123.206.197.235 nc %h %p' 192.168.199.63 "sudo tcpdump -vv -i eth0 port 80 -w -" | wireshark -k -i -

#git diff对比commit差异，只列出文件名
git diff --name-status 128e72a28ea38d1c8691a22a28937356e7adf736^ 128e72a28ea38d1c8691a22a28937356e7adf736

#docker log-opt
--log-driver syslog --log-opt tag="eos-producer"
--log-driver json-file --log-opt max-size=500m

sudo docker run --log-driver=awslogs --log-opt awslogs-region=ap-southeast-1 --log-opt awslogs-group=eos-log --log-opt awslogs-datetime-format='\[%b %d, %Y %H:%M:%S\]'  -d  -p 8898:80 nginx

docker run -d -p 9876:9876 -p 80:8888 --name eos --log-driver gelf --log-opt gelf-address=udp://10.188.10.145:15155  -v /data/eos/:/eos  eoslaomao/eos:1.3.0 /opt/eosio/bin/nodeos --data-dir=/eos/data --config-dir=/eos/conf

docker run -d -p 9876:9876 -p 8888:8888 --name eos --log-driver json-file --log-opt max-size=1g  -v /data/eos/:/eos gcr.io/eosasia-testnet/eos:v1.5.3-patch /opt/eosio/bin/nodeos --data-dir=/eos/data --config-dir=/eos/conf

#pigz多线程打包
apt install pigz
tar --use-compress-program=pigz -cvpf package.tgz ./package
tar --use-compress-program=pigz -xvpf package.tgz -C ./package

#aws elb reg/unreg instance
aws elbv2 register-targets --target-group-arn=arn:aws:elasticloadbalancing:ap-northeast-1:908572518:targetgroup/e-api-1/65c430c260d1556e --targets Id=i-00b7f498a742cd
aws elbv2 deregister-targets --target-group-arn=arn:aws:elasticloadbalancing:ap-northeast-1:908572518:targetgroup/e-api-1/65c430c260d1556e --targets Id=i-00b7f498a742cd

#pmm
docker run -d \
   -p 80:81 \
   --volumes-from pmm-data \
   --name pmm-server \
   --restart always \
   percona/pmm-server:latest

#git 使用指定的private key clone
GIT_SSH_COMMAND="ssh -i ~/.ssh/id_rsa" git clone git@cd.i.ly.com:ly/cat.git


#K8S 端口转发
kubectl -n test port-forward $(kubectl -n test get endpoints prometheus -o jsonpath='{.subsets[0].addresses[0].targetRef.name}') 9090:9090

#jp 处理数据
jq  '.members[]|{"id": .id, "Name": .fullName}' trello.json
jq  '.cards[]|if .idList == "5c64e37e57ba1c20f62751ce" then .idMembers else .xxx end' trello.json

#获取websocket返回信息
s = new WebSocket('wss://www.staging.strikingly.com/livechat-ws')
s.onclose = (e) => { console.log(e)}

# 获取postgresql scheme
select * from information_schema.schemata;

#查看表字段 - postgresql
SELECT a.attname as name, a.attnotnull as notnull
FROM pg_class as c,pg_attribute as a
where c.relname = 'mini_program_accounts' and a.attrelid = c.oid and a.attnum>0

#postgresql dump database
pg_dump -U vi -d vi -h rm-j.pg.rds.aliyuncs.com -p 3433 --no-owner --no-privileges -f vi.sql

#mysqldump
mysqldump --host rm-j.mysql.rds.aliyuncs.com -u vi -p --databases via --column-statistics=0 > vi.sql

# 获取当前会话总数
select count(*) from pg_stat_activity;

# 杀死所有idle的进程
select pg_terminate_backend(pid) from pg_stat_activity where state='idle';

# 查看所有表的空间占用情况
SELECT  
schemaname as schema,  
tablename as table_name,  
pg_size_pretty(pg_relation_size(schemaname || '.' || tablename)) AS size_p,  
pg_total_relation_size(schemaname || '.' || tablename) AS siz,  
pg_size_pretty(pg_total_relation_size(schemaname || '.' || tablename)) AS 表总大小,  
pg_total_relation_size(schemaname || '.' || tablename) - pg_relation_size(schemaname || '.' || tablename) AS 索引大小,  
(100*(pg_total_relation_size(schemaname || '.' || tablename) - pg_relation_size(schemaname || '.' || tablename)))/CASE WHEN pg_total_relation_size(schemaname || '.' || tablename) = 0 THEN 1 ELSE pg_total_relation_size(schemaname || '.' || tablename) END || '%' AS index_pct  
FROM pg_tables  
ORDER BY siz DESC  
```