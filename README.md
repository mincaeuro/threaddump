#!/bin/bash
Servername=$(hostname);
PID=`ps axf | grep 'tomcat/standard/conf/logging.properties' | grep -v grep | awk '{print $1}'`
TMPFILE=~/jstack_$PID_$(date +%d%m%Y_%H_%M)_$Servername.out

clear
for i in 1 2 3 4 5 6
do
echo "############### Threaddump LOOP $i In progress `date` ###############"
echo "################ Threaddump  LOOP $i JSTACK `date` ################" >> $TMPFILE


jstack -l $PID >> $TMPFILE
sleep 30
done
echo "zipping.."
gzip $TMPFILE

gettime=$(date +%d%m%y%H%M);
starttime=$(date +%d%m%y)"1559";

echo $gettime
echo $starttime

if [ $gettime -ge $starttime ]; then

	echo "it's time!!!"
	tar -cvf jstack_$gettime.tar jstack_$(date +%d%m%Y)*
	echo "Hallo, Threaddump aus $Servername im Anhang" | mail -s "Threaddump aus $Servername $(date +%d%m%y)" -r "my@email.com" -a "jstack_$gettime.tar" my@mail.com

else

	echo "No, Not yet!!!"

fi

