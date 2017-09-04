create Cronjob:
```
crontab -e


0 11,13,16 * * 1-5  ~/threaddump.sh >> ~/jstack.log 2>&1

```
crate script file:
```
vim threaddump.sh
```

paste code:
```
#!/bin/bash

Servername=$(hostname);
testtime=$(date +%d%m%Y_%H_%M)
PID=`ps axf | grep 'tomcat/standard/conf/logging.properties' | grep -v grep | awk '{print $1}'`
TMPFILE=/home/user/jstack_$PID_$(date +%d%m%Y_%H_%M)_$Servername.out

clear
for i in 1 2 3 4 5 6
do
echo "############### Threaddump for pid:$PID LOOP $i In progress `date` ###############"
echo "################ Threaddump for pid:$PID  LOOP $i JSTACK `date` ################" >> $TMPFILE


jstack -l $PID >> $TMPFILE
sleep 30
done

echo "checking log.."
result=$(less $TMPFILE | grep "java.lang.Thread.State: BLOCKED")
countresult=$(less $TMPFILE | grep -c "java.lang.Thread.State: BLOCKED")
echo "Nr of Results:"
echo $countresult
echo $result >> /home/user/result_file_$PID_$testtime_$Servername.txt;

echo "zipping.."
gzip $TMPFILE

gettime=$(date +%d%m%y%H%M);
#starttime=$(date +%d%m%y)"1559";
recipients=my@email.com,others@email.com

echo $gettime
echo $starttime

if [ $countresult -ne 0 ]; then

        echo "it's time!!! sending email"
        tar -cvf jstack_$gettime.tar jstack_$(date +%d%m%Y)* result_file_*;
        rm /home/arsystem/jstack_$(date +%d%m%Y)_*;
        echo "Hallo, Threaddump aus $Servername im Anhang. Anzhal von "Blocked Threads": $countresult ." | mail -s "Threaddump aus $Servername $(date +%d%m%y)" -r "my@email.com" -a "jstack_$gettime.tar" $recipients;

rm /home/arsystem/result_file_*;
find /home/arsystem/ -type f -name "jstack_*" -mtime +2 -exec rm {} \;

else

        echo "No, Not yet!!! No Blocked Threads found"
rm /home/arsystem/result_file_*;
find /home/arsystem/ -type f -name "jstack_*" -mtime +2 -exec rm {} \;
fi
```
and make it executable:
```
chmod +x threaddump.sh
```
