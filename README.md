# -telnet-
用telnet批量测试通信IP和端口的连通性
批量检测端口通信：

telnet.sh 脚本内容如下：

文件说明

     telnet_alive.txt  : 活动的端口

     telnet_die.txt    :  离线的端口

    telnet_result.txt  :  活动端口返回信息

    telnet_info.txt     :  要查询的iP端口地址

    telnet.sh            : telnet 查询IP的脚本

 

telnet.sh 的详细信息：

#!/bin/bash

BASEDIR=`dirname $0`
BASEDIR=`cd $BASEDIR;pwd`

result_dir=$BASEDIR/result

telnet_info=telnet_info.txt

for line in `cat $BASEDIR/$telnet_info`
do
   ip=`echo $line | awk 'BEGIN{FS="|"} {print $1}'`
   port=`echo $line | awk 'BEGIN{FS="|"} {print $2}'`
   echo "(sleep 1;) | telnet $ip $port"
   (sleep 1;) | telnet $ip $port > $result_dir/telnet_result.txt
   successIp=`cat $result_dir/telnet_result.txt | grep -B 1 \] | grep [0-9] | awk '{print $3}' | cut -d '.' -f 1,2,3,4`
   if [ -n "$successIp" ]; then
      echo "$successIp|$port" >> $result_dir/telnet_alive.txt
   fi
done

cat $BASEDIR/$telnet_info $result_dir/telnet_alive.txt | sort | uniq -u > $result_dir/telnet_die.txt

 

备注：
telnet.sh 脚本内容说明如下：

#!/bin/bash
#功能，批量telnet端口，输入参数需要测试的IP:PORT列表文件：telnet_list.txt（文件名可以自定义，但是只能跟脚本放在同一目录）
#使用方法： telnet.sh telnet_list.txt ;或者后台执行： sh telnet.sh telnet_list.txt >tellog.log 2>&1 &
#输出2个文件到result目录中： telnet_alive.txt 为端口通的；telnet_die.txt为端口不通的情况。
#文件内容格式如下，文件中每一行第一个字符#开头的行为注释行，不进行处理：
#127.0.0.1|631

#获取当前目录
BASEDIR=`dirname $0`
BASEDIR=`cd $BASEDIR;pwd`

#设置输出数据目录。
mkdir -p $BASEDIR/result
result_dir=$BASEDIR/result


#设置输入的IP和端口文件名
telnet_list=$1
#如果输入参数为空，默认list文件为当前目录下的telnet_list.txt
if [[ -z $telnet_list ]];
then
    echo "=>list file name is default!"
    telnet_list=telnet_list.txt
fi

echo "telnet test file is :  $telnet_list"

#重置上次执行的文件结果
mv $result_dir/telnet_alive.txt $result_dir/telnet_alive.txt.bak

#进行telnet并输出到响应文件中
for line in `cat $BASEDIR/$telnet_list |grep -v ^# |grep -v ^$ `
do
                #获取测试IP
        ip=`echo $line | awk 'BEGIN{FS="|"} {print $1}'`
        #获取测试端口
        port=`echo $line | awk 'BEGIN{FS="|"} {print $2}'`
        #telnent一次并暂停1秒输出到result/telnet_result.txt 文件中，文件数据每一次循环会重置。
        echo "(sleep 1;) | telnet $ip $port"
        (sleep 1;) | telnet $ip $port > $result_dir/telnet_result.txt
        #查找成功响应的数据并输出到到result/telnet_alive.txt 文件中。
        successIp=`cat $result_dir/telnet_result.txt | grep -B 1 \] | grep [0-9] | awk '{print $3}' | cut -d '.' -f 1,2,3,4`
        if [ -n "$successIp" ]; then
                echo "$successIp|$port" >> $result_dir/telnet_alive.txt
        fi
done
#查找失败数据并输出到result/telnet_die.txt文件内。
cat $BASEDIR/$telnet_list $result_dir/telnet_alive.txt | sort | uniq -u |grep -v ^# > $result_dir/telnet_die.txt
