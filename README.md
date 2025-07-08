# ev-charging-slot
Packages
from flask import Flask, render_template, Response, redirect, request, session, abort, url_for
import os
import base64
import mysql.connector
import hashlib
import datetime
from datetime import date
import time
from random import seed
from random import randint
from PIL import Image
import stepic
import urllib.request
import urllib.parse
Database Connection
mydb = mysql.connector.connect(
host=&quot;localhost&quot;,
user=&quot;root&quot;,
password=&quot;&quot;,
charset=&quot;utf8&quot;,
database=&quot;electric_vehicle&quot;
app = Flask(__name__)
##session key
app.secret_key = &#39;abcdef&#39;
Login
def login():
msg=&quot;&quot;
if request.method==&#39;POST&#39;:
uname=request.form[&#39;uname&#39;]
pwd=request.form[&#39;pass&#39;]
cursor = mydb.cursor()
cursor.execute(&#39;SELECT * FROM ev_register WHERE uname = %s AND pass = %s&#39;,
(uname, pwd))
account = cursor.fetchone()
if account:
session[&#39;username&#39;] = uname
return redirect(url_for(&#39;userhome&#39;))
else:
msg = &#39;Incorrect username/password! or access not provided&#39;
return render_template(&#39;login.html&#39;,msg=msg)
User Register
def register():
msg=&quot;&quot;
mycursor = mydb.cursor()
mycursor.execute(&quot;SELECT max(id)+1 FROM ev_register&quot;)
maxid = mycursor.fetchone()[0]
if maxid is None:
maxid=1
if request.method==&#39;POST&#39;:
address=request.form[&#39;address&#39;]
name=request.form[&#39;name&#39;]
mobile=request.form[&#39;mobile&#39;]
email=request.form[&#39;email&#39;]
account=request.form[&#39;account&#39;]
card=request.form[&#39;card&#39;]
bank=request.form[&#39;bank&#39;]
uname=request.form[&#39;uname&#39;]
pass1=request.form[&#39;pass&#39;]
cursor = mydb.cursor()
sql = &quot;INSERT INTO
ev_register(id,name,address,mobile,email,account,card,bank,amount,uname,pass) VALUES
(%s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s)&quot;
val = (maxid,name,address,mobile,email,account,card,bank,&#39;10000&#39;,uname,pass1)
cursor.execute(sql, val)
mydb.commit()
print(cursor.rowcount, &quot;Registered Success&quot;)
msg=&quot;sucess&quot;
return redirect(url_for(&#39;login&#39;))
return render_template(&#39;register.html&#39;,msg=msg)
Charging Slot
def slot():
msg=&quot;&quot;
act=&quot;&quot;
if &#39;username&#39; in session:
uname = session[&#39;username&#39;]
sid=request.args.get(&#39;sid&#39;)
cursor = mydb.cursor()
cursor.execute(&quot;SELECT * FROM ev_station where id=%s&quot;,(sid, ))
dd= cursor.fetchone()
station=dd[1]
cursor.execute(&quot;SELECT * FROM ev_booking where station=%s and status=1&quot;,(sid, ))
data= cursor.fetchall()
for nn in data:
if nn[5]==1:
s1=1
if nn[5]==2:
s2=2
if nn[5]==3:
s3=3
if nn[5]==4:
s4=4
if nn[5]==5:
s5=5
act=&quot;ok&quot;
return
render_template(&#39;slot.html&#39;,msg=msg,uname=uname,sid=sid,station=station,act=act,data=data
,s1=s1,s2=s2,s3=s3,s4=s4,s5=s5,s6=s6,s7=s7,s8=s8,s9=s9,s10=s10)
def select():
if &#39;username&#39; in session:
uname = session[&#39;username&#39;]
sid=request.args.get(&#39;sid&#39;)
rid=request.args.get(&#39;rid&#39;)
if request.method==&#39;POST&#39;:
plan=request.form[&#39;plan&#39;]
cursor = mydb.cursor()
cursor.execute(&quot;update ev_booking set plan=%s,charge_st=1,charge_min=0,charge_sec=0
where id=%s&quot;,(plan, rid))
mydb.commit()
return redirect(url_for(&#39;slot&#39;,sid=sid))
return render_template(&#39;select.html&#39;,sid=sid,rid=rid)
Booking Slot
def book():
msg=&quot;&quot;
act=&quot;&quot;
if &#39;username&#39; in session:
uname = session[&#39;username&#39;]
sid=request.args.get(&#39;sid&#39;)
slot=request.args.get(&#39;slot&#39;)
#if request.method==&#39;GET&#39;:
#sid=request.args.get(&#39;sid&#39;)
#slot=request.args.get(&#39;slot&#39;)
cursor = mydb.cursor()
cursor.execute(&quot;SELECT * FROM ev_station where id=%s&quot;,(sid, ))
dd= cursor.fetchone()
station=dd[1]
#cursor.execute(&quot;SELECT * FROM ev_booking where station=%s and status=1&quot;,(sid, ))
#data= cursor.fetchall()
if request.method==&#39;POST&#39;:
carno=request.form[&#39;carno&#39;]
reserve=request.form[&#39;reserve&#39;]
sid=request.form[&#39;sid&#39;]
slot=request.form[&#39;slot&#39;]
mycursor = mydb.cursor()
mycursor.execute(&quot;SELECT max(id)+1 FROM ev_booking&quot;)
maxid = mycursor.fetchone()[0]
if maxid is None:
maxid=1
t = time.localtime()
rtime = time.strftime(&quot;%H:%M:%S&quot;, t)
today= date.today()
rdate= today.strftime(&quot;%d-%m-%Y&quot;)
rn=randint(1, 10)
cimage=&quot;c&quot;+str(rn)+&quot;.jpg&quot;
cursor = mydb.cursor()
sql = &quot;INSERT INTO
ev_booking(id,uname,station,carno,reserve,slot,cimage,rtime,rdate,status) VALUES (%s, %s,
%s, %s, %s, %s, %s, %s, %s, %s)&quot;
val = (maxid,uname,sid,carno,reserve,slot,cimage,rtime,rdate,&#39;1&#39;)
cursor.execute(sql, val)
mydb.commit()
print(cursor.rowcount, &quot;Booked Success&quot;)
return redirect(url_for(&#39;slot&#39;,sid=sid))
return render_template(&#39;book.html&#39;,msg=msg,uname=uname,sid=sid,slot=slot)
def history():
msg=&quot;&quot;
if &#39;username&#39; in session:
uname = session[&#39;username&#39;]
cursor = mydb.cursor()
cursor.execute(&quot;SELECT * FROM ev_booking b,ev_station s where b.station=s.id and
b.uname=%s&quot;,(uname, ))
data= cursor.fetchall()
return render_template(&#39;history.html&#39;,msg=msg, data=data, uname=uname)
Charging
def charge2():
msg=&quot;&quot;
if &#39;username&#39; in session:
uname = session[&#39;username&#39;]
amt=0
rid=request.args.get(&#39;rid&#39;)
cursor = mydb.cursor()
cursor.execute(&quot;SELECT * FROM ev_booking where id=%s&quot;,(rid, ))
dd= cursor.fetchone()
cmin=dd[17]
csec=dd[18]
print(&quot;sc=&quot;+str(csec))
plan=dd[8]
if plan==1:
cost=100
elif plan==2:
cost=200
else:
cost=300
if csec&lt;60:
csec+=1
cursor = mydb.cursor()
cursor.execute(&quot;update ev_booking set charge_min=%s,charge_sec=%s where
id=%s&quot;,(cmin,csec,rid))
mydb.commit()
else:
cursor = mydb.cursor()
cursor.execute(&quot;update ev_booking set
charge_st=3,charge_time=30,charge_min=%s,charge_sec=%s where id=%s&quot;,(cmin,csec,rid))
mydb.commit()
if dd[19]==3:
amt=cost+dd[15]
cursor = mydb.cursor()
cursor.execute(&quot;update ev_booking set charge_st=4,charge=%s where id=%s&quot;,(amt,rid))
mydb.commit()
return render_template(&#39;charge2.html&#39;,rid=rid, cmin=cmin, csec=csec)
Payment
def payment ():
if &#39;username&#39; in session:
uname = session[&#39;username&#39;]
amount=0
rid=request.args.get(&#39;rid&#39;)
sid=request.args.get(&#39;sid&#39;)
cursor = mydb.cursor()
cursor.execute(&quot;SELECT * FROM ev_register where uname=%s&quot;,(uname, ))
uu= cursor.fetchone()
card=uu[6]
cursor.execute(&quot;SELECT * FROM ev_booking where id=%s&quot;,(rid, ))
dd= cursor.fetchone()
amt=dd[15]
ch=dd[15]
t = time.localtime()
rtime = time.strftime(&quot;%H:%M:%S&quot;, t)
today= date.today()
rdate= today.strftime(&quot;%d-%m-%Y&quot;)
if ch&gt;0:
amount=ch
else:
amount=20
cursor = mydb.cursor()
cursor.execute(&quot;update ev_booking set edate=%s,etime=%s,amount=%s where
id=%s&quot;,(rdate,rtime,amount,rid))
mydb.commit()
if request.method==&#39;POST&#39;:
pay_mode=request.form[&#39;pay_mode&#39;]
if pay_mode==&quot;Bank&quot;:
rn=randint(1000, 9999)
otp=str(rn)
cursor = mydb.cursor()
cursor.execute(&quot;update ev_booking set pay_mode=%s,sms_st=1,otp=%s where
id=%s&quot;,(pay_mode,otp,rid))
mydb.commit()
return redirect(url_for(&#39;verify_otp&#39;,rid=rid))
else:
cursor = mydb.cursor()
cursor.execute(&quot;update ev_booking set pay_mode=%s,pay_st=1 where
id=%s&quot;,(pay_mode,rid))
mydb.commit()
return redirect(url_for(&#39;slot&#39;,sid=sid))
return render_template(&#39;payment.html&#39;,sid=sid,rid=rid,
uname=uname,amount=amount,card=card)
def verify_otp():
msg=&quot;&quot;
if &#39;username&#39; in session:
uname = session[&#39;username&#39;]
amount=0
rid=request.args.get(&#39;rid&#39;)
sid=request.args.get(&#39;sid&#39;)
cursor = mydb.cursor()
cursor.execute(&quot;SELECT * FROM ev_register where uname=%s&quot;,(uname, ))
uu= cursor.fetchone()
mobile=uu[3]
cursor.execute(&quot;SELECT * FROM ev_booking where id=%s&quot;,(rid, ))
dd= cursor.fetchone()
key=dd[14]
amount=dd[15]
sms_st=dd[22]
if request.method==&#39;POST&#39;:
otp=request.form[&#39;otp&#39;]
if key==otp:
if sms_st==1:
message=&quot;OTP:&quot;+otp
print(&quot;sent&quot;+str(mobile))
cursor = mydb.cursor()
cursor.execute(&quot;update ev_booking set pay_st=2,sms_st=0 where
id=%s&quot;,(pay_mode,otp,rid))
mydb.commit()
#cursor.execute(&quot;update ev_register set amount=amount-%s where
uname=%s&quot;,(amount,uname))
#mydb.commit()
#return redirect(url_for(&#39;slot&#39;,sid=sid))
msg=&quot;Amount Paid Successfully&quot;
return render_template(&#39;verify_otp.html&#39;,rid=rid,sid=sid,msg=msg)
