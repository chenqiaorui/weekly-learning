### shell模板
#### 部署服务
```
[ ! "$USER" == "www" ]&&{ echo "Please switch user 'www' to run deploy.sh";exit 1; }

# Branch=test Project=web-project Target_dir_develop=/tmp
# test web-pr /tmp/website/web-pro machainIP
Branch=$1
Project=$2
Build_dir="/tmp/${Project}/${Branch}"
Archive_file="${Project}.tar.gz"
Build_history_list="${Build_dir}/history_list.txt"
Target_dir_develop=$3

Lock_file="/tmp/${Project}-${Branch}.lock"
Log_file="/tmp/${Project}-${Branch}.log"

shift 3
HOSTGROUP="$*"
Ssh_port=8022

ENV=develop
INPUT=deploy

echo "$0 $1 $2 $3"

TIMESTAMP=`date +%F_%T`

#echo "$USER@$HOSTNAME to execute deploy.sh $1 $2 at ${TIMESTAMP}"

writelog(){
    if [ $1 == 'true' ]
    then
        echo "${TIMESTAMP} -- $USER@$HOSTNAME to execute deploy.sh ${INPUT} ${ANY_VERSION}" >> ${Log_file}
    elif [ $1 == 'false' ]
    then
        echo "failure -- ${TIMESTAMP} -- $USER@$HOSTNAME to execute deploy.sh ${INPUT} ${ANY_VERSION}" >> ${Log_file}
    else
        echo "${TIMESTAMP} -- $*" >> ${Log_file}
    fi
}

# 5.
archive_build(){
    grep "\<$1\>" $Build_history_list > /dev/null
    if [ $? -eq 0 ]
    then
        cd $Build_dir
        echo "<INFO>    Arching package <$1> as ${Archive_file} ..."
        tar zcf ${Archive_file} $1 || return 1
        export Unpack_file=$1
        return 0
    else
        writelog `echo "Can not find the last version in ${Build_dir},Please check histroy_list have last version"`
        return 1
    fi
}
archive_once(){
    if [ "$1" == "latest" ]
    then
        archive_build `tail -n1 ${Build_history_list}`
        [ $? -eq 0 ]&&return 0 || return 1

    elif [ "$1" == "last" ]
    then
        archive_build `tac ${Build_history_list}|sed -n 2p`
        [ $? -eq 0 ]&&return 0 || return 1
    # 4.always here
    else
        archive_build $1
        [ $? -eq 0 ]&&return 0 || return 1
    fi
}

scp_file(){
    #if [ -z "$1" ]
#    if [ "$1" == "latest" ]
#    then
#       archive_build `tail -n1 ${Build_history_list}`
#
#    elif [ "$1" == "last" ]
#    then
#       archive_build `tac ${Build_history_list}|sed -n 2p`
#    else
#       archive_build $1
#    fi

#    [ $? -eq 0 ]||return 1
#deploy
    echo "<INFO>        Distribute package to $ip"
    if [ "$ENV" == "develop" ]
    then
        scp -P ${Ssh_port} ${Archive_file} ${User_host}:${Target_dir_develop} > /dev/null && \
        [ $? -ne 0 ]&& \
        { writelog `echo "scp failed, please check the server"`;return 1; }
            writelog `echo "scp ${Archive_file} to ${User_host}:${Target_dir_develop} Successful"`
    elif [ "$ENV" == "product" ]
    then
        scp -P ${Ssh_port} ${Archive_file} ${User_host}:${Target_dir_product} && \
            writelog `echo "scp ${Archive_file} to ${User_host}:${Target_dir_product} Successful"`
    elif [ "$ENV" == "all" ]
    then
        scp -P ${Ssh_port} ${Archive_file} ${User_host}:${Target_dir_develop} && \
            scp -P ${Ssh_port} ${Archive_file} ${User_host}:${Target_dir_product}
        [ $? -eq 0 ]&& \
            writelog `echo "scp ${Archive_file} to ${User_host}:${Target_dir_develop} and ${Target_dir_product} Successful"`
    elif [ -z "$ENV" ]
    then
        scp -P ${Ssh_port} ${Archive_file} ${User_host}:${Target_dir_develop} && \
            writelog `echo "scp ${Archive_file} to ${User_host}:${Target_dir_develop} Successful"`
    else
        writelog `echo "Uneable to scp ${Archive_file} to Target host."`
        return 1
    fi

    return 0
}

update_code(){
    #ssh -p ${Ssh_port} ${User_host} ||{ writelog `echo "Can't ssh to ${User_host}"`;exit 1; }
    if [ "$ENV" == "product" ]
    then
        ssh -p ${Ssh_port} ${User_host} \
            "cd ${Target_dir_product};rm -fr ${Unpack_file} &> /dev/null;tar xf ${Archive_file} && { rm build;ln -s ${Unpack_file} build; }"
        [ $? -eq 0 ]&& \
            { writelog `echo "update link 'build' --> '${Unpack_file}' at $ENV Successful"`;return 0; }

    elif [ "$ENV" == "all" ]
    then
        ssh -p ${Ssh_port} ${User_host} \
            "cd ${Target_dir_develop};rm -fr ${Unpack_file} &> /dev/null;tar xf ${Archive_file} && { rmbuild;ln -s ${Unpack_file} build; } && \
            { cd ${Target_dir_product};rm -fr ${Unpack_file} &> /dev/null;tar xf ${Archive_file}; } && {rm build;ln -s ${Unpack_file} build;}"
        [ $? -eq 0 ]&& \
            { writelog `echo "update link 'build' --> '${Unpack_file}' at develop and product Successful"`;return 0; }

    elif [ "$ENV" == "develop" ]
    then
        ssh -p ${Ssh_port} ${User_host} \
            "cd ${Target_dir_develop};rm -fr ${Unpack_file} &> /dev/null;tar xf ${Archive_file} && { rm build;ln -s ${Unpack_file} build; }"
        [ $? -eq 0 ]&& \
            { writelog `echo "update link 'build' --> '${Unpack_file}' at $ENV Successful "`;return 0; } || { writelog `echo "last step failed, can't unpack file"`;return 1; }
    else
        writelog `echo "Uneable to create link to ${Unpack_file}"`
        return 1
    fi
}

# 2.
deploy(){
    # 3.
    archive_once latest
    [ $? -eq 0 ]|| { echo 'archive failed.';exit 1; }
    for ip in $HOSTGROUP
    do
        User_host=www@$ip
                ssh -p ${Ssh_port} ${User_host} "[ -d ${Target_dir_develop} ]||mkdir ${Target_dir_develop}"
        scp_file && update_code &
    done
    [ $? -eq 0 ] && return 0 || return 1
}

rollback_any(){
    archive_once ${ANY_VERSION}
    for ip in $HOSTGROUP
    do
        User_host=www@$ip
        scp_file && update_code &
    done
    [ $? -eq 0 ] && return 0 || return 1
}

rollback_emergency(){
    archive_once last
    [ $? -eq 0 ]&&return 0 || { echo 'archive failed.';exit 1; }
    for ip in $HOSTGROUP
    do
        User_host=www@$ip
        scp_file && update_code &
    done
    [ $? -eq 0 ] && return 0 || return 1
}

main(){
# 1.文件存在
[ -e $Lock_file ] && { echo "The $0 has been locked"; exit 1; } || (touch $Lock_file)
    # INPUT always deploy
    case ${INPUT} in
        deploy)
                deploy;
                ;;
        rollback_emergency)
                rollback_emergency;
                ;;
        rollback_any)
                rollback_any;
                ;;
        *)
            echo 'usage: deploy.sh (devlop|product|all) (deploy|rollback_emergency|rollback_any) [any_Version]';
            rm $Lock_file;
            exit 1;
    esac;
    [ $? -eq 0 ]&&{ wait;rm $Lock_file;echo "done.";exit 0; }||{ wait;rm $Lock_file;exit 1; }
}

main
```

#### 重启服务
#!/bin/bash

DIR=$PWD
SERVICE_NAME=te_emog
/opt/go/bin/go build -o build/$SERVICE_NAME > /dev/null || echo 'go build step error!'

cd $PWD/supervisor
[ -e supervisor.sock ] && supervisorctl restart all || supervisord -c supervisord.conf

================or=================

#!/bin/bash
DIR=$PWD
BRANCH=$(basename -s ref`cat .git/HEAD`)
PROJECT=$(basename $PWD)
NAME=$PROJECT-$BRANCH

/opt/go/bin/go build -o build/$NAME &> /dev/null||echo 'go get code.aliyun.com...'

export GOPROXY=https://goproxy.io
export GO111MODULE=on

#PID=`/usr/sbin/lsof cmd|grep $NAME|awk '{print $2}'`
PID=`ps -ef|grep -v grep|grep $NAME|awk '{print $2}'`
[ $PID ] && { kill $PID; while true; do [ -e /proc/$PID/stat ] || break; done; }
nohup build/$NAME -c config/service.yaml &> logs/output &
sleep 2
cat logs/output


#### 并发执行脚本
```
#!/bin/bash

{ echo 'p1开始执行: '`date`; sleep 5; echo 'p1执行结束'`date`; } &
{ echo 'p2开始执行: '`date`; sleep 5; echo 'p2执行结束'`date`; } &
wait

echo '上面并发执行完，轮到打印我'
```

=======or===========

```
#!/bin/bash
beginTime=`date +%s`
num=1
for i in `seq 1 3`
do
	{	
       	echo $i  "业务逻辑 开始执行,当前时间:" `date "+%Y-%m-%d %H:%M:%S"`
		sleep 2s
		echo $i  "业务逻辑 执行完成,当前时间:" `date "+%Y-%m-%d %H:%M:%S"`
		echo "-----------------------------------------------------------"
	# 结尾的&确保每个进程后台执行
	}&
done
# wait关键字确保每一个子进程都执行完成
wait
endTime=`date +%s`
echo "总共耗时:" $(($endTime-$beginTime)) "秒"
```