---
title: "Shell 动态进程守护"
date: 2020-02-11T14:21:06+08:00
tags: [ "运维" ]
---

主要用于openwrt里面的进程守护的脚本，检测程序是否运行，程序不存在就运行相应的脚本
<!--more-->

### 配置文件
``` uci
config general general
    option enable 1

config monitor
    option name 'scplc'
    option enable 1
    option pschkstr 'sc-serv plc'
    option type     'application'
    option tmpfile '/tmp/monitor/plc.mon'
    option mqttkey '/app/mon/plc/status'
    option timeLimit 60 
    option execcmd ''

config monitor
    option name 'scgpio'
    option enable 1
    option pschkstr 'sc-serv gpio'
    option type     'application'
    option tmpfile '/tmp/monitor/gpio.mon'
    option mqttkey '/app/mon/gpio/status'
    option timeLimit 60 
    option execcmd ''

```
### 代码
```sh
flash="0"
true="1"
false="0";
disable="0"
enable="1"
configName="scmonitor"
printHelp(){
	clear
	echo "USAGE:"
	echo "-g <monitor type> get current moitor enable status "
	echo "-l view current all montoir list"
	echo "-s <monitorName> <monitorValue> set monitor state"
	echo "-a autoManage "
	echo "-e enable cron"
	echo "-d disable cron"
	echo "add <name> <enable> <pschkstr> <sevicename> <tmpfile> <mqttkey> <execcmd> <timeLimit>"
  echo "example:"
	echo "	check -l"
	echo "			plc"
	echo "			pjlink"
	echo "   check -g plc"
	echo "			current monitor state:0"
	echo "   check -s plc 0"
}

#检测线程是否存在
function_thread_check(){
	#count= ${ps |grep 'sc-serv plc' | grep -v 'grep' | wc -l}
	count=$( ps | grep "$*" | grep -v 'grep' | wc -l )
	if [ 0 == $count ];then
		echo false
	else
		echo true
	fi
}

#获取当前所有模块的名称
function_getMonitorList(){
	count=$(uci get $configName.general.count)

	for i in $(seq 0 $count);
	do
		name=$(uci get $configName.@monitor[$i].name)
		echo $name

	done
}
#获取状态
function_getMonitorStats(){
    count=$(uci get $configName.general.count)

    for i in $(seq 0 $count);
    do
		name=$(uci get $configName.@monitor[$i].name)
        if [ "$name" == "$1" ]; then
            state=$(uci get $configName.@monitor[$i].enable)
            echo $1 stats is $state
        fi
    done
}
#获取当前的时间差
function_getTimeCompare(){
	currTime=$( date +%s)
	lastTime=$1
	#tmpTime= $("$currTime - $lastTime")

	echo $(($currTime - $lastTime))
}

function_setMonitorState(){
	count=$(uci get $configName.general.count)
	for i in $(seq 0 $count);
	do
		name=$(uci get $configName.@monitor[$i].name)
        old_sate=$(uci get $configName.@monitor[$i].enable)
		if [ "$name" == "$1" ]; then
			state=$(uci set $configName.@monitor[$i].enable=$2)
            echo "$1 set success"
		fi
	done
}

# 自动化处理流程
function_autoManage(){
	count=$(uci get $configName.general.count)
	monitorState=$(uci get $configName.general.enable)

	if [ $monitorState == "0" ]; then
		echo "current server monitor disable"
		return 
	fi

	for i in $(seq 0 $count);
	do
		name=$(uci get $configName.@monitor[$i].name)
        psChkstr=$(uci get $configName.@monitor[$i].pschkstr)
		tmpfile=$(uci get $configName.@monitor[$i].tmpfile)
        type=$(uci get $configName.@monitor[$i].type)
		result=$(function_thread_check $psChkstr)
		MonitorStats=$(uci get $configName.@monitor[$i].enable)

		if [ "$MonitorStats" == "$disable" ]; then 
			echo  $name current Current status is disable 
			continue	
		fi

		if [ "$result" == "false" ]; then
      echo "Services have been shut down, which is restarting"
			#execCmd=$(uci get sc-monitor.@monitor[$i].execcmd)
			$(/etc/init.d/$name start)
		else
      if [ $type == "application" ]; then #application不做文件检测
          echo "$name Operating normally"
          continue
      fi

			if [ ! -e "$tmpfile" ]; then
				$(/etc/init.d/$name stop)
				$(/etc/init.d/$name start)
				echo "restart $name server"
                continue
			else
				lastTime=$(cat $tmpfile)
				timeLimit=$(uci get $configName.@monitor[$i].timeLimit)
				result=$(function_getTimeCompare $lastTime )

				if [ "$result" -gt "$timeLimit" ]; then  #如果当前返回的值大于允许值
					$(rm -rf $tmpfile)
					$(/etc/init.d/$name stop)
					$(/etc/init.d/$name start)
				fi

				echo $result $timeLimit
			fi

		fi
        echo "$name Operating normally"
	done
}

#移除服务
function_removeMonitor(){
	MonitorName=$1
	tmp=$(uci get $configName.general.count)
	for i in $(seq 0 $tmp);
	do
		name=$(uci get $configName.@monitor[$i].name)
		if [ "$name" == "$MonitorName" ]; then
			count=$((  $tmp - 1 ))
			$(uci delete $configName.@monitor[$i])
			$(uci set $configName.general.count=$count)
			$(uci commit $configName)
		fi
	done
}

function_EnableCron(){
	$(uci set $configName.general.enable=1)
	$(uci commit $configName)
}

function_DisableCron(){
	$(uci set $configName.general.enable=1)
	$(uci commit $configName)
}

[ "$(cat /proc/uptime | awk -F. '{print $1}')" -gt 300 ] || exit 0

if [ "$#" -eq 0 ]; then
	printHelp
	exit 1
fi

if [ "$1" == "-s" ]; then
	if [  "$2" == "" ] || [ "$3" == "" ]; then
		exit 1
	fi
	function_setMonitorState $2 $3
	exit 1

fi

if [ "$1" == "add" ]; then
	if [ "$2" == "" ]; then
		echo "plese input type"
		exit 1
	fi
	if [ "$3" == "" ]; then
		echo "plese input eanbled"
		exit 1
	fi

	if [ "$4" == "" ]; then
		echo "plese input pschkstr"
		exit 1
	fi
	if [ "$5" == "" ]; then
		echo "plese input sevicename "
		exit 1
	fi
	if [ "$6" == "" ]; then
		echo "plese input tmpfile"
		exit 1
	fi
	if [ "$7" == "" ]; then
		echo "plese input mqttkey"
		exit 1
	fi
	if [ "$8" == "" ]; then
		echo "plese input execcmd"
		exit 1
	fi
	if [ "$9" == "" ]; then
		echo "plese input timeLimit"
		exit 1
	fi
	tmp=$(uci get $configName.general.count)
	#$count=$(( $count + 1 ))
	count=$((  $tmp + 1 ))

	$(uci add $configName monitor)
	$(uci add_list $configName.@monitor[$count].name="$2")
	$(uci add_list $configName.@monitor[$count].enable="$3")
	$(uci add_list $configName.@monitor[$count].pschkstr="$4")
	$(uci add_list $configName.@monitor[$count].servicename="$5")
	$(uci add_list $configName.@monitor[$count].tmpfile="$6")
	$(uci add_list $configName.@monitor[$count].mqttkey="$7")
	$(uci add_list $configName.@monitor[$count].execcmd="$8")
	$(uci add_list $configName.@monitor[$count].timeLimit="$9")
	$(uci set $configName.general.count=$count)
	$(uci commit $configName)
	exit 1
fi

while getopts g:s:lear:d opt
do
	case "$opt" in
		g)
		function_getMonitorStats $OPTARG
		;;
		e)
		function_EnableCron
		;;
		d)	function_DisableCron
		;;
		l)
		function_getMonitorList
		;;
		r)
		function_removeMonitor $OPTARG
		;;
		a)
		function_autoManage
		;;
		?)
		clear
		printHelp ;;
	esac
done

```