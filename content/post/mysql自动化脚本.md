---
title: "mysql 运维脚本"
date: 2020-02-11T14:15:06+08:00
tags: [ "运维" ]
---

把mysql常用操作通过shell进行封装
<!--more-->
``` js
#!/bin/ash


USER="root"
NEWUSER="xxx"
PASSWORD="xxxx"
NEWPASSWORD="xxxxxxxxxx"
DATE=`date +%Y/%m/%d`
OUTPUT="/mnt/data/database/$DATE"
OPERATOR="export"
FILES="/tmp/database/*"
DATABASEPATH="/mnt/db/mysql"
TMPPATH="/mnt/db/tmp"
echo $OUTPUT
[ ! -d "$OUTPUT" ] && {
    echo "mkdir $OUTPUT"
    mkdir -p $OUTPUT
}
#数据为初始化
action_init(){
    flag=$(uci get -q system.@general[0].databaseinit)
    [ ! -z "$flag" ] && return
	[  -z "$flag" ] && {
        section=$(uci get -q system.@general[0])
		[ -z "$section" ] &&{
			uci add system general
		}
		uci set system.@general[0].databaseinit=1
        uci commit system
	}
    [ -d "$DATABASEPATH" ] && rm -rf $DATABASEPATH
    [ -d "$TMPPATH" ] && rm -rf $TMPPATH
    sed -i "s,^datadir.*,datadir         = \"$DATABASEPATH\",g" /etc/my.cnf
    sed -i 's,^tmpdir.*,tmpdir          = "/mnt/db/tmp",g' /etc/my.cnf
    [ ! -d "$DATABASEPATH" ] && mkdir -p $DATABASEPATH
    [ ! -d "$TMPPATH" ] && mkdir -p $TMPPATH
    mysql_install_db --force
    /etc/init.d/mysqld start
    /etc/init.d/mysqld enable
    mysqladmin -u  $USER password "$PASSWORD"
    [ -f /tmp/adduser.sql ] && rm -rf /tmp/adduser.sql
    echo "insert into mysql.user(Host,User,Password) values(\"localhost\",\"$NEWUSER\",password(\"$NEWPASSWORD\"));">/tmp/adduser.sql
    echo "GRANT ALL PRIVILEGES ON *.* TO \"$NEWUSER\"@\"localhost\" IDENTIFIED BY \"$NEWPASSWORD\"; "
    echo "GRANT ALL PRIVILEGES ON *.* TO \"$NEWUSER\"@\"localhost\" IDENTIFIED BY \"$NEWPASSWORD\"; ">>/tmp/adduser.sql
    echo "flush privileges;">>/tmp/adduser.sql
    mysql -u $USER -p$PASSWORD < /tmp/adduser.sql
    rm -rf /tmp/adduser.sql
}
#数据库导出
action_export(){
    databases=`mysql -u $USER -p$PASSWORD -e "SHOW DATABASES;" | tr -d "| " | grep -v Database`
    echo "backup start"
    for db in $databases; do
        if [[ "$db" != "information_schema" ]] && [[ "$db" != "performance_schema" ]] && [[ "$db" != "mysql" ]] && [[ "$db" != _* ]] ; then
            echo "Dumping database: $db"
            [ -f $OUTPUT/`date +%Y%m%d`.$db.sql ] && {
                echo "rm sql file"
                rm -rf $OUTPUT/`date +%Y%m%d`.$db.sql
            }
            mysqldump -u $NEWUSER -p$NEWPASSWORD --databases $db > $OUTPUT/`date +%Y%m%d`.$db.sql
            [ -f $OUTPUT/`date +%Y%m%d`.$db.sql.gz ] && {
                echo "rm file"
                rm -rf $OUTPUT/`date +%Y%m%d`.$db.sql.gz
            }
            gzip $OUTPUT/`date +%Y%m%d`.$db.sql
            rm -rf $OUTPUT/`date +%Y%m%d`.$db.sql
        fi
    done
    echo "backup end"
}
#数据库导入
action_import(){
    for f in $FILES
    do
        [ -f "$f" ] &&{
            echo "Processing $f file..."
            mysql -u $NEWUSER -p$NEWPASSWORD < $f
        }
    done
}
[ ! -z "$1" ] && OPERATOR=$1
[ ! -z "$2" ] && USER=$2
[ ! -z "$3" ] && PASSWORD=$3


if [ "$OPERATOR" == "export" ]; then
    action_export
elif [ "$OPERATOR" == "import" ]; then
    action_import
elif [ "$OPERATOR" == "init" ]; then
    action_init
fi


```