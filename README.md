import tensorflow as tf
import random

from flask import Flask, render_template, request, session, jsonify,json
from DBConnection import Db
from PIL import Image, ImageOps
app = Flask(__name__)
# import tensorflow as tf
app.secret_key ="abcd"


static_path="C:\\Users\\91884\\PycharmProjects\\retinal_disease\\static\\"


@app.route('/ab')
def hello_world():
    return 'Hello World!'



#----------------------------ADMIN

@app.route('/')
def login():
    return render_template("login_index.html")


@app.route('/login_post', methods=['post'])
def login_post():
    username = request.form['textfield']
    password = request.form['textfield2']
    d = Db()
    qry = "SELECT * FROM `login` WHERE `uname`='"+username+"' AND `password`='"+password+"'"
    res = d.selectOne(qry)
    if res!=None:
        session['lid']=res['lid']
        if res['type']=='admin':
            return admin_index()
        elif res['type']=='hospital':
            return  hosp_index()
        elif res['type']=='doctor':
            return doctor_index()
        elif res['type']=='user':
            return user_index()
        else:
            return login()
    else:

        return login()



@app.route('/admin_index')
def admin_index():
    return render_template("admin/admin_index.html")


@app.route('/approved_hospital')
def approved_hospital():
    qry="SELECT * FROM`login`,`hospital` WHERE `login`.`lid`=`hospital`.`hlid` AND`login`.`type`='hospital'"

    db=Db()
    res=db.select(qry)
    return render_template("admin/approved hospital.html",data=res)

@app.route('/approve_hospital/<id>')
def approve_hospital(id):
    db=Db()
    res="UPDATE `login` SET `type`='hospital' WHERE lid='"+id+"'"
    qry=db.insert(res)
    return '''<script>alert('Approved');window.location='/pending_hospital'</script>'''




@app.route('/pending_hospital')
def pending_hospital():
    res="select * from hospital,login where hospital.hlid=login.lid and login.type='pending'"
    db=Db()
    qry=db.select(res)
    return render_template("admin/pending hospital.html",data=qry)



@app.route('/rejected_hospital')
def rejected_hospital():
    res="select * from hospital,login where hospital.hlid=login.lid and login.type='rejected'"
    db=Db()
    qry=db.select(res)
    return render_template("admin/rejected hospital.html",data=qry)

@app.route('/rejected_hospitals/<id>')
def rejected_hospitals(id):
    db=Db()
    res="UPDATE`login` SET`type`='rejected' WHERE lid='"+id+"'"
    qry=db.insert(res)
    return '''<script>alert('Rejected');window.location='/pending_hospital'</script>'''

@app.route('/reply_complaint/<id>')
def reply_complaint(id):
    return render_template("admin/reply to complaint.html",id=id)


@app.route('/reply_compliant_post',methods=['post'])
def reply_compliant_post():
    id=request.form['aa']
    reply=request.form['textarea']
    db=Db()
    qry = "UPDATE `complaint` SET reply='"+reply+"' where cid='"+str(id)+"'"
    res=db.update(qry)
    print(res)
    return '''<script>alert('Reply Send');window.location='/view_complaint'</script>'''

@app.route('/view_complaint')
def view_complaint():
    res="select * from complaint,user where complaint.ulid=user.ulid and reply='pending'"
    db=Db()
    qry=db.select(res)
    return render_template("admin/view complaint.html",data=qry)



@app.route('/change_passwordad')
def change_passwordad():
    return render_template("admin/chnage password.html")


@app.route('/change_passwordad_post',methods=['post'])
def change_passwordad_post():
    oldpass=request.form['textfield']
    passw=request.form['textfield2']
    conpass=request.form['textfield3']
    db = Db()
    qry="SELECT * FROM `login` WHERE PASSWORD='"+oldpass+"' AND lid='"+str(session['lid'])+"'"
    res=db.selectOne(qry)
    if res is not None:
        if passw == conpass:
            qry1 = "UPDATE `login` SET `password`='" + passw + "' WHERE password='" + oldpass + "' AND lid='" + str(session['lid']) + "'"
            res1 = db.update(qry1)
            return '''<script>alert('Password Changed Successfully');window.location='/change_passwordad'</script>'''
        else:
            return '''<script>alert('Check Password Typed');window.location='/change_passwordad'</script>'''
    else:
        return '''<script>alert('User Not Found');window.location='/change_passwordad'</script>'''




#-----------ACCURACY


@app.route('/v')
def v():
    steps = [0, 100, 200, 300, 400, 500, 600, 700, 800, 900, 1000, 1100, 1200, 1300.1400, 1500, 1600, 1700, 1800, 1900,
             2000, 2100, 2200, 2300, 2400, 2500,2600,2700,2800, 2900, 3000, 3100, 3200, 3300, 3400, 3500, 3600, 3700,
             3800, 3900, 4000, 4100, 4200, 4300.4400, 4500, 4600, 4700, 4800, 4900, 4999]
    train = [57.0, 89.0, 83.0, 81.0, 91.0, 87.0, 92.0, 83.0, 91.0, 95.0, 90.0, 92.0, 91.0, 92.0, 95.0, 97.0, 97.0, 99.0,
             97.0, 91.0, 96.0, 99.0, 91.0, 97.0, 93.0, 94.0, 93.0, 96.0, 99.0, 95.0, 98.0, 96.0, 100, 99.0, 93.0, 97.0,
             98.0, 98.0, 96.0, 100, 97.0, 98.0, 98.0, 99.0, 98.0, 98.0, 99.0, 98.0]
    validation = [50.0, 76.0, 79.0, 76.0, 80.0, 77.0, 77.0, 81.0, 76.0, 77.0, 81.0, 85.0, 83.0, 81.0, 78.0, 78.0, 86.0,
                  78.0, 82.0, 77.0, 83.0, 84.0, 80.0, 79.0, 89.0, 83.0, 81.0, 78.0, 79.0, 83.0, 81.0, 74.0, 73.0, 79.0,
                  80.0, 84.0, 83.0, 85.0, 83.0, 85.0, 77.0, 71.0, 73.0, 78.0, 83.0, 81.0, 84.0, 73.0, 77.0, 81.0, 74.0]

    return render_template("admin/trial.html",train=train,val=validation,steps=steps,l=len(train))


@app.route('/v1')
def v1():
    steps = [0, 100, 200, 300, 400, 500, 600, 700, 800, 900, 1000.1100, 1200, 1300, 1400, 1500, 1600, 1700, 1800, 1900,
             2000, 2100, 2200, 2300, 2400, 2500, 2600, 2700, 2800, 2900, 3000, 3100, 3200, 3300, 3400, 3500, 3600, 3700,
             3800, 3900,
             4000, 4100, 4200, 4300, 4400, 4500, 4600, 4700, 4800, 4900, 4999]
    train = [30.0, 60.0, 73.0, 72.0, 61.0, 72.0, 72.0, 66.0, 72.0, 67.0, 71.0, 71.0, 69.0, 70.0, 67.0, 73.0, 74.0, 71.0,
             73.0, 76.0, 72.0, 76.0, 75.0, 72.0, 76.0,
             80.0, 77.0, 75.0, 75.0, 72.0, 71.0, 75.0, 78.0, 80.0, 83.0, 78.0, 76.0, 79.0, 84.0, 82.0, 84.0, 78.0, 73.0,
             84.0, 81.0, 79.0, 87.0, 78.0, 74.0]
    validation = [14.0, 64.0, 58.0, 56.0, 65.0, 53.0, 60.0, 60.0, 63.0, 56.0, 58.0, 39.0, 61.0, 54.0, 55.0, 52.0, 57.0,
                  64.0, 67.0,
                  61.0, 62.0, 64.0, 70.0, 63.0, 60.0, 75.0, 71.0, 54.0, 60.0, 63.0, 62.0, 60.0, 54.0, 53.0, 63.0, 73.0,
                  68.0, 66.0, 69.0,
                  63.0, 59.0, 64.0, 58.0, 60.0, 65.0, 62.0, 71.0, 75.0, 66.0]


    return render_template("admin/trial1.html",train=train,val=validation,steps=steps,l=len(train))


@app.route('/v2')
def v2():
    steps = [0, 100, 200, 300, 400, 500, 600, 700, 800, 900, 1000.1100, 1200, 1300, 1400, 1500, 1600, 1700, 1800, 1900,
             2000, 2100, 2200, 2300, 2400, 2500, 2600, 2700, 2800, 2900, 3000, 3100, 3200, 3300, 3400, 3500, 3600, 3700,
             3800, 3900,
             4000, 4100, 4200, 4300, 4400, 4500, 4600, 4700, 4800, 4900, 4999]
    train = [54.0, 81.0, 82.0, 75.0, 77.0, 84.0, 84.0, 87.0, 80.0, 79.0, 84.0, 85.0, 87.0, 87.0, 89.0, 85.0, 91.0, 87.0,
             91.0, 91.0, 93.0, 91.0,
             94.0, 92.0, 97.0, 94.0, 90.0, 90.0, 87.0, 97.0, 93.0, 95.0, 96.0, 94.0, 94.0, 92.0, 93.0, 94.0, 96.0, 98.0,
             94.0, 97.0, 94.0, 99.0, 93.0, 96.0, 93.0, 96.0, 96.0, 92.0]
    validation = [45.0, 70.0, 71.0, 69.0, 70.0, 74.0, 74.0, 75, 75.0, 85.0, 76.0, 76.0, 71.0, 79.0, 76.0, 77.0, 73.0,
                  81.0, 75.0, 78.0, 82.0, 73.0, 80.0, 85.0, 71.0, 78.0, 79.0, 77.0,
                  75.0, 73.0, 81.0, 76.0, 86.0, 68.0, 78.0, 82.0, 73.0, 82.0, 73.0, 82.0, 76.0, 83.0, 82.0, 78.0, 76.0,
                  69.0, 75.0, 73.0, 73.0, 71.0]

    return render_template("admin/trial2.html",train=train,val=validation,steps=steps,l=len(train))










#-------------------------DOCTOR
#
# @app.route('/home_doctor')
# def home_doctorl():
#     return render_template("doctor/doctor_homepage.html")


@app.route('/doctor_index')
def doctor_index():
    return render_template("doctor/doctor_index.html")






@app.route('/booking/<id>')
def booking(id):
    res="SELECT * FROM `booking`,`user` WHERE slid='"+str(id)+"' AND `booking`.`ulid`=`user`.`ulid`"
    db=Db()
    qry=db.select(res)
    return render_template("doctor/booking.html",data=qry)

# @app.route('/edit_profile')
# def edit_profile():
#     return render_template("doctor/edit profile.html")

# @app.route('/image_upload')
# def image_upload():
#     return render_template("doctor/image upload.html")

@app.route('/scheduling')
def scheduling():
    res="select * from schedule,doctors where schedule.dlid=doctors.dlid and schedule.dlid='"+str(session['lid'])+"' "
    db=Db()
    qry=db.select(res)
    return render_template("doctor/Scheduling.html",data=qry)


@app.route('/shedule_searh')
def shedule_searh():
    qry="SELECT * FROM  schedule WHERE dlid='"+str(session['lid'])+"'"
    db=Db()
    res=db.select(qry)
    return render_template("doctor/Scheduling.html",data=res)

@app.route('/shedule_searh_post',methods=['post'])
def shedule_searh_post():
    frm=request.form['textfield']
    to=request.form['textfield2']
    qry="SELECT * FROM `schedule` WHERE sdate BETWEEN '"+frm+"' and'"+to+"' AND dlid='"+str(session['lid'])+"' "
    db=Db()
    res=db.select(qry)
    return render_template("doctor/Scheduling.html",data=res)


@app.route('/view_profile')
def view_profile():
    db=Db()
    qry="SELECT * FROM `doctors` WHERE `dlid`='"+str(session['lid'])+"'"
    res=db.selectOne(qry)
    return render_template("doctor/view profile.html",data=res)





@app.route('/change_passworddr')
def change_passworddr():
    return render_template("doctor/chnage password.html")



@app.route('/change_passworddr_post',methods=['post'])
def change_passworddr_post():
    oldpass=request.form['textfield']
    passw=request.form['textfield2']
    conpass=request.form['textfield3']
    db = Db()
    qry="SELECT * FROM `login` WHERE PASSWORD='"+oldpass+"' AND lid='"+str(session['lid'])+"'"
    res=db.selectOne(qry)
    if res is not None:
        if passw == conpass:
            qry1 = "UPDATE `login` SET `password`='" + passw + "' WHERE password='" + oldpass + "' AND lid='" + str(session['lid']) + "'"
            res1 = db.update(qry1)
            return '''<script>alert('Password Changed Successfully');window.location='/change_passworddr'</script>'''
        else:
            return '''<script>alert('Check Password Typed');window.location='/change_passworddr'</script>'''
    else:
        return '''<script>alert('User Not Found');window.location='/change_passworddr'</script>'''










#-----------------------HOSPITAL

# @app.route('/home_hospital')
# def home_hospital():
#     return render_template("hospital/hospital_homepage.html")

@app.route('/hosp_index')
def hosp_index():
    return render_template("hospital/hospial_index.html")



@app.route('/add_dr')
def add_dr():
    return render_template("hospital/add dr.html")

@app.route('/add_dr_post',methods=['post'])
def add_dr_post():
    name=request.form['textfield']
    gender=request.form['gender']
    email=request.form['textfield2']
    # contact=request.form['textfield3']
    place=request.form['textfield4']
    post=request.form['textfield5']
    pin=request.form['textfield6']
    qualification=request.form['textfield7']
    experience=request.form['textfield8']
    image = request.files['fileField']
    image.save(static_path + 'doctor\\' + image.filename)
    path = '/static/doctor/' + image.filename
    password=random.randint(0000,9999)
    db = Db()
    qry = "INSERT INTO login(`uname`,`password`,`type`)VALUES('" + email + "','"+ str(password) +"','doctor')"
    res = db.insert(qry)
    qry1 = "INSERT INTO `doctors`(`dlid`,`hlid`,`name`,`gender`,`email`,`place`,`post`,`pin`,`qualification`,`experience`,`image`)VALUES('" + str(
        res) + "','"+str(session['lid'])+"','" + name + "','"+gender+"','" + email + "','" + place + "','" + post + "','"+pin+"','" + qualification + "','"+experience+"','" + str(path)+ "')"
    res1 = db.insert(qry1)
    return '''<script>alert('Added');window.location='/add_dr'</script>'''

@app.route('/add_dr_schedule')
def add_dr_schedule():
    res="SELECT * FROM `doctors` WHERE hlid='"+str(session['lid'])+"'"
    db=Db()
    qry=db.select(res)
    return render_template("hospital/add dr schedule.html",data=qry)

@app.route('/add_schedule_post',methods=['post'])
def add_schedule_post():
    dr=request.form['select']
    fromtime= request.form['textfield2']
    totime=request.form['textfield3']
    date=request.form['textfield4']
    qry="INSERT INTO `schedule`(`dlid`,`sdate`,`time from`,`time to`) VALUES('"+dr+"','"+date+"','"+fromtime+"','"+totime+"')"
    db=Db()
    res=db.insert(qry)
    return '''<script>alert('Added');window.location='/add_dr_schedule'</script>'''


@app.route('/change_password')
def change_password():
    return render_template("hospital/chnage password.html")

@app.route('/change_password_post',methods=['post'])
def change_passworddr_post_post():
    oldpass=request.form['textfield']
    passw=request.form['textfield2']
    conpass=request.form['textfield3']
    db = Db()
    qry="SELECT * FROM `login` WHERE PASSWORD='"+oldpass+"' AND lid='"+str(session['lid'])+"'"
    res=db.selectOne(qry)
    if res is not None:
        if passw == conpass:
            qry1 = "UPDATE `login` SET `password`='" + passw + "' WHERE password='" + oldpass + "' AND lid='" + str(session['lid']) + "'"
            res1 = db.update(qry1)
            return '''<script>alert('Password Changed Successfully');window.location='/change_password'</script>'''
        else:
            return '''<script>alert('Check Password Typed');window.location='/change_password'</script>'''
    else:
        return '''<script>alert('User Not Found');window.location='/change_password'</script>'''


@app.route('/dr_management')
def dr_management():
    res="select * from doctors"
    db=Db()
    qry=db.select(res)
    return render_template("hospital/dr managment.html",data=qry)

@app.route('/edit_dr_details/<id>')
def edit_dr_details(id):
    db = Db()
    qry = "SELECT * FROM doctors WHERE dlid='"+id + "'"
    res = db.selectOne(qry)
    session["iid"]=id
    return render_template("hospital/edit dr details.html", data=res)


@app.route('/edit_dr_details_post',methods=['post'])
def edit_dr_details_post():
    name = request.form['textfield']
    gender = request.form['gender']
    email = request.form['textfield2']
    place = request.form['textfield4']
    post = request.form['textfield5']
    pin = request.form['textfield6']
    qualification = request.form['textfield7']
    experience = request.form['textfield8']
    db = Db()
    if 'fileField' in request.files:
        image = request.files['fileField']
        if image.filename != "":
            image.save(static_path + 'doctor\\' + image.filename)
            path = '/static/doctor/' + image.filename
            qry1 = "UPDATE `doctors` SET `name`='" + name + "',`gender`='" + gender + "', `email`='" + email + "',`place`='" + place + "', `post`='" + post + "', `pin`='" + pin + "', `qualification`='" + qualification + "',`experience`='" + experience + "', image ='" + path + "'  WHERE `dlid`='" + str(session['iid']) + "'"
            res1 = db.update(qry1)
            return '''<script>alert('Changes Added');window.location='/view_drs'</script>'''
        else:
            qry1 = "UPDATE `doctors` SET `name`='" + name + "',`gender`='" + gender + "',email='" + email + "',`place`='" + place + "', `post`='" + post + "', `pin`='" + pin + "', `qualification`='" + qualification + "',`experience`='" + experience + "'   WHERE `dlid`='" + str( session['iid']) +"'"
            res1 = db.update(qry1)
            return '''<script>alert('Changes Added');window.location='/view_drs'</script>'''
    else:
        qry1 = "UPDATE `doctors` SET `name`='" + name + "',`gender`='" + gender + "',email='" + email + "',`place`='" + place + "', `post`='" + post + "', `pin`='" + pin + "', `qualification`='" + qualification + "',`experience`='" + experience + "'  WHERE `dlid`='" + str(session['iid']) + "'"
        res1 = db.update(qry1)
        return '''<script>alert('Changes Added');window.location='/view_drs'</script>'''



@app.route('/edit_profile2')
def edit_profile2():
    db = Db()
    qry = "SELECT *FROM `hospital` WHERE hlid='" + str(session['lid']) + "'"
    res = db.selectOne(qry)
    return render_template("hospital/edit profile.html",data=res)

@app.route('/edit_profile2_post',methods=['post'])
def edit_profile2_post():
    name = request.form['textfield']
    place = request.form['textfield1']
    post = request.form['textfield2']
    pin = request.form['textfield3']
    contact = request.form['textfield4']
    email = request.form['textfield5']
    licence=request.form['textfield6']
    db = Db()
    if 'fileField' in request.files:
        image = request.files['fileField']
        if image.filename != "":
            image.save(static_path + 'hospital\\' + image.filename)
            path = '/static/hospital/' + image.filename
            qry1 = "UPDATE `hospital` SET`name`='"+name+"',place='"+place+"',post='"+post+"',pin='"+pin+"',contact='"+contact+"',email='"+email+"',licence='"+licence+"',image ='"+path+"'  WHERE `hlid`='" + str(session['lid']) + "' "
            res1 = db.update(qry1)
            return '''<script>alert('Changes Added');window.location='/view_profile3'</script>'''
        else:
            qry1 = "UPDATE `hospital` SET`name`='" + name + "',place='" + place + "',post='" + post + "',pin='" + pin + "',contact='" + contact + "',email='" + email + "',licence='" + licence + "'  WHERE `hlid`='" + str(session['lid']) + "' "
            res1 = db.update(qry1)
            return '''<script>alert('Changes Added');window.location='/view_profile3'</script>'''
    else:
        qry1 = "UPDATE `hospital` SET`name`='" + name + "',place='" + place + "',post='" + post + "',pin='" + pin + "',contact='" + contact + "',email='" + email + "',licence='" + licence + "'  WHERE `hlid`='" + str(
            session['lid']) + "' "
        res1 = db.update(qry1)
        return '''<script>alert('Changes Added');window.location='/view_profile3'</script>'''




@app.route('/edit_schedule/<id>')
def edit_schedule(id):
    db = Db()
    qry = "SELECT *FROM `schedule` WHERE sid='"+str(id)+"'"
    res = db.selectOne(qry)
    session["iid"] = id
    return render_template("hospital/edit schedule.html", data=res)

@app.route('/edit_schedule_post',methods=['post'])
def edit_schedule_post():
    fromtime=request.form['textfield2']
    totime=request.form['textfield3']
    date=request.form['textfield4']
    id=str(session["iid"])
    db=Db()
    qry="UPDATE schedule SET `sdate`='"+date+"' ,`time from`='"+fromtime+"' ,`time to`='"+totime+"'  WHERE `sid`='"+str(id)+"' "
    res=db.update(qry)
    return '''<script>alert('Changes Added');window.location='/view_dr_schedule'</script>'''


@app.route('/signup')
def signup():
    return render_template("hospital/signup_hospital.html")

@app.route('/signup_post',methods=['post'])
def signup_post():
    name=request.form['textfield']
    place=request.form['textfield6']
    post=request.form['textfield7']
    pin=request.form['textfield8']
    contact=request.form['textfield4']
    email=request.form['textfield5']
    license=request.form['textfield2']
    image=request.files['fileField']
    image.save(static_path + 'hospital\\'+ image.filename)
    path='/static/hospital/' + image.filename
    db=Db()
    qry="INSERT INTO login(`uname`,`password`,`type`)VALUES('"+email+"','"+contact+"','pending')"
    res=db.insert(qry)
    qry1="INSERT INTO hospital(`hlid`,`name`,`place`,`post`,`pin`,`contact`,`email`,`licence`,`image`)VALUES('"+str(res)+"','"+name+"','"+place+"','"+post+"','"+pin+"','"+contact+"','"+email+"','"+license+"','"+path+"')"
    res1=db.insert(qry1)
    return render_template('login_index.html')



@app.route('/view_booking/<id>')
def view_booking(id):
    qry="SELECT * FROM `booking`,`schedule`,`user`WHERE `booking`.slid='"+str(id)+"'AND `schedule`.sid='"+str(id)+"' AND`booking`.ulid=`user`.ulid"
    db=Db()
    res=db.select(qry)
    return render_template("hospital/view booking.html",data=res)

@app.route('/view_dr_schedule')
def view_dr_schedule():
    qry = "select * from doctors,schedule WHERE `hlid`='" + str(session['lid']) + "' AND schedule.dlid=doctors.dlid"
    db = Db()
    res = db.select(qry)
    s = "select * from doctors WHERE `hlid`='" + str(session['lid']) + "'"
    dd = db.select(s)
    return render_template("hospital/view dr schedule.html", data=res, dd=dd)

app.route('/drshed_searh')
def drshed_searh():
    qry="SELECT * FROM  `schedule`,`doctors` WHERE schedule.dlid=doctors.dlid,"
    db=Db()
    res=db.select(qry)
    s="select * from doctors WHERE `hlid`='" + str(session['lid']) + "'"
    dd=db.select(s)
    return render_template("hospital/view dr schedule.html",data=res,dd=dd)

@app.route('/drshed_searh_post',methods=['post'])
def  drshed_searh_post():
    drname=request.form['select']
    qry="SELECT * FROM  `schedule`,`doctors` WHERE schedule.dlid=doctors.dlid  AND doctors.dlid='"+drname+"'"
    db=Db()
    res=db.select(qry)
    s = "select * from doctors WHERE `hlid`='" + str(session['lid']) + "'"
    dd = db.select(s)
    return render_template("hospital/view dr schedule.html", data=res, dd=dd)


@app.route('/delete_sched/<id>')
def delete_sched(id):
    d = Db()
    qry = "DELETE FROM `schedule` WHERE `sid`='"+str(id)+"'"
    res = d.delete(qry)
    return view_dr_schedule()





@app.route('/view_drs')
def view_drs():
    qry="select * from doctors  WHERE `hlid`='"+str(session['lid'])+"'"
    db=Db()
    res=db.select(qry)
    return render_template("hospital/view drs.html",data=res)

app.route('/drs_searh')
def drs_searh():
    qry="SELECT * FROM doctors"
    db=Db()
    res=db.select(qry)
    return render_template("hospital/view drs.html",data=res)

@app.route('/drs_searh_post',methods=['post'])
def  drs_searh_post():
    drname=request.form['textfield']

    qry="SELECT * FROM `doctors` WHERE name LIKE '%"+drname+"%'"
    db=Db()
    res=db.select(qry)
    return render_template("hospital/view drs.html",data=res)



@app.route('/delete_dr/<id>')
def delete_dr(id):
    d = Db()
    qry = "DELETE FROM doctors WHERE `dlid`='"+str(id)+"'"
    res = d.delete(qry)
    return view_drs()



@app.route('/view_profile3')
def view_profile3():
    db = Db()
    qry = "SELECT * FROM `hospital` WHERE `hlid`='" + str(session['lid']) + "'"
    res = db.selectOne(qry)
    return render_template("hospital/view profile.html", data=res)







#------------------------------USER

# @app.route('/user_hospital')
# def user_hospital():
#     return render_template("user/user_homepage.html")

@app.route('/user_index')
def user_index():
    return render_template("user/user_index.html")




@app.route('/signup2')
def signup2():
    return render_template("user/signup_user.html")


@app.route('/signup2_post',methods=['post'])
def signup2_post():
    name = request.form['textfield']
    gender = request.form['gender']
    dateofbirth=request.form['textfield3']
    contact=request.form['textfield4']
    email=request.form['textfield5']
    place = request.form['textfield6']
    post = request.form['textfield7']
    pin = request.form['textfield8']
    image = request.files['fileField']
    image.save(static_path + 'user\\' + image.filename)
    path = '/static/user/' + image.filename
    password = random.randint(0000, 9999)
    db = Db()
    qry = "INSERT INTO login(`uname`,`password`,`type`)VALUES('" + email + "','" + str(password) + "','user')"
    res = db.insert(qry)
    qry1 = "INSERT INTO `user`(`ulid`,`name`,`gender`,`dob`,`place`,`post`,`pin`,`contact`,`email`,`image`)VALUES('" + str( res) + "','" + name + "','" + gender + "','"+dateofbirth+"','" + place + "','" + post + "','" + pin + "','" + contact + "','" + email + "','" + str(   path) + "')"
    res1 = db.insert(qry1)
    return render_template('login_index.html')







@app.route('/booking_history')
def booking_history():
    res="SELECT * FROM `booking`,`schedule`,`doctors` WHERE `booking`.slid=`schedule`.sid AND `booking`.ulid='"+str(session['lid'])+"' AND `doctors`.dlid=`schedule`.dlid"
    db=Db()
    qry=db.select(res)
    return render_template("user/booking history.html",data=qry)


@app.route('/bookinguser/<id>')
def bookinguser(id):
    qry="INSERT INTO `booking`(`slid`,`ulid`,`date`)VALUES('"+str(id)+"','"+str(session['lid'])+"',curdate())"
    db=Db()
    res=db.insert(qry)
    return '''<script>alert('Booked');window.location='/view_doctors'</script>'''




@app.route('/change_password2')
def change_password2():
    return render_template("user/chnage password.html")

@app.route('/change_password2_post',methods=['post'])
def change_password2_post():
    oldpass=request.form['textfield']
    passw=request.form['textfield2']
    conpass=request.form['textfield3']
    db = Db()
    qry="SELECT * FROM `login` WHERE PASSWORD='"+oldpass+"' AND lid='"+str(session['lid'])+"'"
    res=db.selectOne(qry)
    if res is not None:
        if passw == conpass:
            qry1 = "UPDATE `login` SET `password`='" + passw + "' WHERE password='" + oldpass + "' AND lid='" + str(session['lid']) + "'"
            res1 = db.update(qry1)
            return '''<script>alert('Password Changed Successfully');window.location='/change_password2'</script>'''
        else:
            return '''<script>alert('Check Password Typed');window.location='/change_password2'</script>'''
    else:
        return '''<script>alert('User Not Found');window.location='/change_password2'</script>'''






@app.route('/complaint')
def complaint():
    return render_template("user/complainnt.html")


@app.route('/complaint_post',methods=['post'])
def complaint_post():
    comp=request.form['textarea']
    db=Db()
    qry="INSERT INTO `complaint`(`ulid`,`complaint`,`date`,`reply`) VALUES ('"+str(session['lid'])+"','"+comp+"',curdate(),'pending')"
    res=db.insert(qry)
    return '''<script>alert('Complaint Sent');window.location='/complaint'</script>'''


@app.route('/view_complain_reply')
def view_complain_reply():
    db = Db()
    qry = "SELECT * FROM `complaint` WHERE ulid='" + str(session['lid']) + "'"
    res = db.select(qry)
    return render_template("user/view reply to complaint.html", data=res)


@app.route('/edit_profile4')
def edit_profile4():
    db = Db()
    qry = "SELECT * FROM `user` WHERE `ulid`='" + str(session['lid']) + "'"
    res = db.selectOne(qry)
    return render_template("user/edit profile.html",data=res)

@app.route('/edit_profile4_post',methods=['post'])
def edit_profile4_post():
    name = request.form['textfield']
    gender = request.form['gender']
    dateofbirth = request.form['textfield3']
    contact = request.form['textfield4']
    email = request.form['textfield5']
    place = request.form['textfield6']
    post = request.form['textfield7']
    pin = request.form['textfield8']
    db = Db()
    if 'fileField' in request.files:

        image = request.files['fileField']
        if image.filename!="":
            image.save(static_path + 'user\\' + image.filename)
            path = '/static/user/' + image.filename

            qry1 ="UPDATE `user` SET`name`='"+name+"',gender='"+gender+"',dob='"+dateofbirth+"',contact='"+contact+"',email='"+email+"',image ='"+path+"', place='"+place+"',post='"+post+"',pin='"+pin+"' WHERE `ulid`='" + str(session['lid']) + "' "
            res1 = db.update(qry1)
            return '''<script>alert('Changes Added');window.location='/view_profile4'</script>'''
        else:
            qry1 ="UPDATE `user` SET`name`='"+name+"',gender='"+gender+"',dob='"+dateofbirth+"',contact='"+contact+"',email='"+email+"', place='"+place+"',post='"+post+"',pin='"+pin+"' WHERE `ulid`='" + str(session['lid']) + "' "
            res1 = db.update(qry1)
            return '''<script>alert('Changes Added');window.location='/view_profile4'</script>'''
    else:
        qry1 = "UPDATE `user` SET`name`='" + name + "',gender='" + gender + "',dob='" + dateofbirth + "',contact='" + contact + "',email='" + email + "', place='" + place + "',post='" + post + "',pin='" + pin + "' WHERE `ulid`='" + str(session['lid']) + "' "
        res1 = db.update(qry1)
        return '''<script>alert('Changes Added');window.location='/view_profile4'</script>'''


@app.route('/image_upload2')
def image_upload2():
    return render_template("user/image upload.html",disease="")

#
# @app.route('/image_upload2_post',methods=['post'])
# def image_upload2_post():
#     img=request.files["fileField"]
#     img.save("C:\\Users\\91884\\PycharmProjects\\retinal_disease\\static\\check_img\\og.jpg")
#
#     # import numpy as np
#     # import cv2 as cv
#     # img = cv.imread("C:\\Users\\91884\\PycharmProjects\\retinal_disease\\static\\check_img\\"+img.filename, 0)
#     # img.save("C:\\Users\\91884\\PycharmProjects\\retinal_disease\\static\\check_img\\a.jpeg")
#     # cv.imshow('C:\\Users\\91884\\PycharmProjects\\retinal_disease\\static\\check_img\\"+img.filename', img)
#     # print(img)
#
#     # GRAY SCALE
#     og_image= Image.open("C:\\Users\\91884\\PycharmProjects\\retinal_disease\\static\\check_img\\og.jpg")
#     gray_image = ImageOps.grayscale(og_image)
#     gray_image.save("C:\\Users\\91884\\PycharmProjects\\retinal_disease\\static\\check_img\\a.jpg")
#     # print(img)
#     path="C:\\Users\\91884\\PycharmProjects\\retinal_disease\\static\\check_img\\"+img.filename
#     path1="C:\\Users\\91884\\PycharmProjects\\retinal_disease\\static\\check_img\\a.jpg"
#
#
#     # --------------------Gaussian Blur
#     import cv2
#     import numpy as np
#
#     # read image
#     src = cv2.imread("C:\\Users\\91884\\PycharmProjects\\retinal_disease\\static\\check_img\\a.jpg", cv2.IMREAD_UNCHANGED)
#     # apply guassian blur on src image
#     dst = cv2.GaussianBlur(src, (5, 5), cv2.BORDER_DEFAULT)
#     # display input and output image
#     cv2.waitKey(0)  # waits until a key is pressed
#     cv2.destroyAllWindows()  # destroys the window showing image
#     cv2.imwrite('C:\\Users\\91884\\PycharmProjects\\retinal_disease\\static\\check_img\\g.jpg',dst)
#
#     #--------- log filter
#
#     # read image
#     img_src = cv2.imread('C:\\Users\\91884\\PycharmProjects\\retinal_disease\\static\\check_img\\og.jpg')
#
#     # kernal sensitive to horizontal lines
#     kernel = np.array([[-1.0, -1.0],
#                        [2.0, 2.0],
#                        [-1.0, -1.0]])
#
#     kernel = kernel / (np.sum(kernel) if np.sum(kernel) != 0 else 1)
#
#     # filter the source image
#     img_rst = cv2.filter2D(img_src, -1, kernel)
#
#     # save result image
#     cv2.imwrite('C:\\Users\\91884\\PycharmProjects\\retinal_disease\\static\\check_img\\filter.jpg', img_rst)
#
#
#     # --------- log filter2
#
#     # read image
#     img_src = cv2.imread('C:\\Users\\91884\\PycharmProjects\\retinal_disease\\static\\check_img\\og.jpg')
#
#     # edge detection filter
#     kernel = np.array([[0.0, -1.0, 0.0],
#                        [-1.0, 4.0, -1.0],
#                        [0.0, -1.0, 0.0]])
#
#     kernel = kernel / (np.sum(kernel) if np.sum(kernel) != 0 else 1)
#
#     # filter the source image
#     img_rst1 = cv2.filter2D(img_src, -1, kernel)
#
#     # save result image
#     cv2.imwrite('C:\\Users\\91884\\PycharmProjects\\retinal_disease\\static\\check_img\\filter2.jpg', img_rst1)
#
#     #---------------CANNY IMAGE
#
#     img1 = cv2.imread("C:\\Users\\91884\\PycharmProjects\\retinal_disease\\static\\check_img\\og.jpg")
#     t_lower = 50  # Lower Threshold
#     t_upper = 150  # Upper threshold
#
#     # Applying the Canny Edge filter
#     edge = cv2.Canny(img1, t_lower, t_upper)
#     print(edge)
#     # img2=cv2.imshow('edge', edge)
#     cv2.waitKey(0)
#     cv2.destroyAllWindows()
#     cv2.imwrite('C:\\Users\\91884\\PycharmProjects\\retinal_disease\\static\\check_img\\b.jpg',edge)
#
#
#     import classify
#     s=classify.abcde()
#     if s=="glaucoma negative":
#         d="Glaucoma Negative"
#     elif s=="glaucoma positive":
#         d="Glaucoma Positive"
#     elif s=="glaucoma positive":
#         d="Glaucoma Positive"
#     elif s=="mild":
#         d=" mild Diabetic Retinopathy"
#     elif s=="moderate":
#         d=" moderate Diabetic Retinopathy"
#     elif s=="no dr":
#         d="No Diabetic Retinopathy"
#     elif s=="proliferate dr":
#         d="proliferate Diabetic Retinopathy"
#     elif s=="severe":
#         d="severe Diabetic Retinopathy"
#     elif s=="amd":
#         d="age-related macular degeneration"
#     elif s=="cataract":
#         d="cataract"
#     elif s=="hypertension":
#         d="Hypertension"
#     elif s=="myopia":
#         d="Myopia"
#
#     return render_template("/image1.html",disease=d)
#



@app.route('/image_upload2_post',methods=['post'])
def image_upload2_post():
    img=request.files["fileField"]
    img.save("C:\\Users\\91884\\PycharmProjects\\retinal_disease\\static\\check_img\\og.jpg")

    # import numpy as np
    # import cv2 as cv
    # img = cv.imread("C:\\Users\\91884\\PycharmProjects\\retinal_disease\\static\\check_img\\"+img.filename, 0)
    # img.save("C:\\Users\\91884\\PycharmProjects\\retinal_disease\\static\\check_img\\a.jpeg")
    # cv.imshow('C:\\Users\\91884\\PycharmProjects\\retinal_disease\\static\\check_img\\"+img.filename', img)
    # print(img)

    # GRAY SCALE
    og_image= Image.open("C:\\Users\\91884\\PycharmProjects\\retinal_disease\\static\\check_img\\og.jpg")
    gray_image = ImageOps.grayscale(og_image)
    gray_image.save("C:\\Users\\91884\\PycharmProjects\\retinal_disease\\static\\check_img\\a.jpg")
    # print(img)
    path="C:\\Users\\91884\\PycharmProjects\\retinal_disease\\static\\check_img\\"+img.filename
    path1="C:\\Users\\91884\\PycharmProjects\\retinal_disease\\static\\check_img\\a.jpg"


    # --------------------Gaussian Blur
    import cv2
    import numpy as np

    # read image
    src = cv2.imread("C:\\Users\\91884\\PycharmProjects\\retinal_disease\\static\\check_img\\a.jpg", cv2.IMREAD_UNCHANGED)
    # apply guassian blur on src image
    dst = cv2.GaussianBlur(src, (5, 5), cv2.BORDER_DEFAULT)
    # display input and output image
    cv2.waitKey(0)  # waits until a key is pressed
    cv2.destroyAllWindows()  # destroys the window showing image
    cv2.imwrite('C:\\Users\\91884\\PycharmProjects\\retinal_disease\\static\\check_img\\g.jpg',dst)

    #--------- log filter

    # read image
    img_src = cv2.imread('C:\\Users\\91884\\PycharmProjects\\retinal_disease\\static\\check_img\\og.jpg')

    # kernal sensitive to horizontal lines
    kernel = np.array([[-1.0, -1.0],
                       [2.0, 2.0],
                       [-1.0, -1.0]])

    kernel = kernel / (np.sum(kernel) if np.sum(kernel) != 0 else 1)

    # filter the source image
    img_rst = cv2.filter2D(img_src, -1, kernel)

    # save result image
    cv2.imwrite('C:\\Users\\91884\\PycharmProjects\\retinal_disease\\static\\check_img\\filter.jpg', img_rst)


    # --------- log filter2

    # read image
    img_src = cv2.imread('C:\\Users\\91884\\PycharmProjects\\retinal_disease\\static\\check_img\\og.jpg')

    # edge detection filter
    kernel = np.array([[0.0, -1.0, 0.0],
                       [-1.0, 4.0, -1.0],
                       [0.0, -1.0, 0.0]])

    kernel = kernel / (np.sum(kernel) if np.sum(kernel) != 0 else 1)

    # filter the source image
    img_rst1 = cv2.filter2D(img_src, -1, kernel)

    # save result image
    cv2.imwrite('C:\\Users\\91884\\PycharmProjects\\retinal_disease\\static\\check_img\\filter2.jpg', img_rst1)

    #---------------CANNY IMAGE

    img1 = cv2.imread("C:\\Users\\91884\\PycharmProjects\\retinal_disease\\static\\check_img\\og.jpg")
    t_lower = 50  # Lower Threshold
    t_upper = 150  # Upper threshold

    # Applying the Canny Edge filter
    edge = cv2.Canny(img1, t_lower, t_upper)
    print(edge)
    # img2=cv2.imshow('edge', edge)
    cv2.waitKey(0)
    cv2.destroyAllWindows()
    cv2.imwrite('C:\\Users\\91884\\PycharmProjects\\retinal_disease\\static\\check_img\\b.jpg',edge)



    type=request.form["select"]
    if type=="Glaucoma":

        import classify_glaucoma
        s,score=classify_glaucoma.abcde()
        if s=="glaucoma negative":
            d="Glaucoma Negative"

        else:
            d="Glaucoma Positive"
    elif type=="Diabetic Retinopathy":
        import classify_dr
        s,score = classify_dr.abcde()
        if s=="mild":
            d=" mild Diabetic Retinopathy"
        elif s=="moderate":
            d=" moderate Diabetic Retinopathy"
        elif s=="no dr":
            d="No Diabetic Retinopathy"
        elif s=="proliferate dr":
            d="proliferate Diabetic Retinopathy"
        else:
            d="severe Diabetic Retinopathy"
    else:
        import classify
        s,score= classify.abcde()
        if s=="amd":
            d="age-related macular degeneration"
        elif s=="cataract":
            d="cataract"
        elif s=="hypertension":
            d="Hypertension"
        else:
            d="Myopia"

    return render_template("/image1.html",disease=d)




@app.route('/view_doctors/<id>')
def view_doctors(id):
    res="SELECT * FROM `doctors` WHERE `hlid`='"+str(id)+"'"
    db=Db()
    qry=db.select(res)
    return render_template("user/view doctor.html",data=qry)

@app.route('/viewdr_searh')
def viewdr_searh():
    qry="SELECT * FROM  doctors"
    db=Db()
    res=db.select(qry)
    return render_template("user/view doctor.html",data=res)

@app.route('/viewdr_searh_post',methods=['post'])
def viewdr_searh_post():
    dname=request.form['textfield']
    qry="SELECT * FROM `doctors` where `name` LIKE'%"+dname+"%'"
    db=Db()
    res=db.select(qry)
    return render_template("user/view doctor.html",data=res)



@app.route('/view_hospital')
def view_hospital():
    res="select * from hospital"
    db=Db()
    qry=db.select(res)
    return render_template("user/View hospital.html",data=qry)

@app.route('/view_profile4')
def view_profile4():
    db = Db()
    qry = "SELECT * FROM `user` WHERE `ulid`='"+str(session['lid'])+"'"
    res = db.selectOne(qry)
    print(res)
    return render_template("user/view profile.html", data=res)

#
# @app.route('/view_reply')
# def view_reply():
#     res="select * from complaint"
#     db=Db()
#     qry=db.select(res)
#     return render_template("user/view reply to complaint.html",data=qry)



@app.route('/view_scheduling2/<id>')
def view_scheduling2(id):
    db=Db()
    qry="SELECT * FROM `schedule`,`doctors` WHERE `schedule`.dlid=`doctors`.dlid AND `schedule`.dlid='"+str(id)+"'"
    res = db.select(qry)
    return render_template("user/view scheduling.html",data=res)

@app.route('/viewshedule_searh')
def viewshedule_searh():
    qry="SELECT * FROM  schedule WHERE dlid='"+str(session['lid'])+"'"
    db=Db()
    res=db.select(qry)
    return render_template("user/view scheduling.html",data=res)

@app.route('/viewshedule_searh_post',methods=['post'])
def viewshedule_searh_post():
    frm=request.form['textfield']
    to=request.form['textfield2']
    qry="SELECT * FROM `schedule` WHERE sdate BETWEEN '"+frm+"' and'"+to+"'"
    db=Db()
    res=db.select(qry)
    return render_template("user/view scheduling.html",data=res)








# ***************************************



if __name__ == '__main__':
 app.run(debug=True,port=5000)
