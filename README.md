# retinal-disease-project

from flask import Flask, render_template, request,session
from DBConnection import Db
from PIL import Image, ImageOps
app = Flask(__name__)

app.secret_key ="h1"


static_path="C:\\Users\\91884\\PycharmProjects\\retinal_disease\\static\\"


@app.route('/ab')
def hello_world():
    return 'Hello World!'



#----------------------------ADMIN

@app.route('/')
def login():
    return render_template("login.html")


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
            return render_template('admin/admin_homepage.html')
        elif res['type']=='hospital':
            return render_template('hospital/hospital_homepage.html')
        elif res['type']=='doctor':
            return render_template('doctor/doctor_homepage.html')
        elif res['type']=='user':
            return render_template('user/user_homepage.html')
        else:
            return '''<script>alert('Invalid');window.location='/'</script>'''
    else:

        return '''<script>alert('Invalid');window.location'/'</script>'''





@app.route('/approved_hospital')
def approved_hospital():
    res="select * from hospital,login where hospital.hlid=login.lid and login.type='approved'"
    db=Db()
    qry=db.select(res)
    return render_template("admin/approved hospital.html",data=qry)

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

@app.route('/reply_complaint')
def reply_complaint():
    return render_template("admin/reply to complaint.html")


@app.route('/reply_compliant_post',methods=['post'])
def reply_compliant_post():
    reply=request.form['textarea']
    qry="insert into complaint(`reply`)values('')"
    db=Db()
    res=db.insert(qry)
    return render_template("admin/view complaint.html")


@app.route('/view_complaint')
def view_complaint():
    res="select * from complaint,user where complaint.ulid=user.ulid and reply='pending'"
    db=Db()
    qry=db.select(res)
    return render_template("admin/view complaint.html",data=qry)



#-------------------------DOCTOR

@app.route('/booking')
def booking():
    res="select * from booking,schedule,user where booking.slid=schedule.sid and booking.ulid=user.ulid"
    db=Db()
    qry=db.select(res)
    return render_template("doctor/booking.html",data=qry)

@app.route('/edit_profile')
def edit_profile():
    return render_template("doctor/edit profile.html")

@app.route('/image_upload')
def image_upload():
    return render_template("doctor/image upload.html")

@app.route('/scheduling')
def scheduling():
    res="select * from schedule,doctors where schedule.dlid=doctors.dlid  "
    db=Db()
    qry=db.select(res)
    return render_template("doctor/Scheduling.html",data=qry)

@app.route('/view_profile')
def view_profile():
    return render_template("doctor/view profile.html")




#-----------------------HOSPITAL

@app.route('/home_hospital')
def home_hospital():
    return render_template("hospital/hospital_homepage.html")

@app.route('/add_dr')
def add_dr():
    return render_template("hospital/add dr.html")

@app.route('/add_dr_post',methods=['post'])
def add_dr_post():
    name=request.form['textfield']
    gender=request.form['gender']
    email=request.form['textfield2']
    contact=request.form['textfield3']
    place=request.form['textfield4']
    post=request.form['textfield5']
    pin=request.form['textfield6']
    qulaification=request.form['textfield7']
    experience=request.form['textfield8']
    image = request.files['fileField']
    image.save(static_path + 'doctor\\' + image.filename)
    path = '/static/doctor/' + image.filename
    db = Db()
    qry = "INSERT INTO login(`uname`,`password`,`type`)VALUES('" + email + "','" + contact + "','hospital')"
    res = db.insert(qry)
    qry1 = "INSERT INTO `doctors`(`dlid`,`hlid`,`name`,`gender`,`email`,`place`,`post`,`pin`,`qualification`,`experience`,`image`)VALUES('" + str(
        res) + "','" + name + "','" + place + "','" + post + "','" + pin + "','" + contact + "','" + email + "','" + license + "','" + path + "')"
    res1 = db.insert(qry1)
    return "ok"


@app.route("/add_dr_schedule")
def add_dr_schedule():
    return render_template("hospital/add dr schedule.html")

@app.route('/change_password')
def change_password():
    return render_template("hospital/chnage password.html")

@app.route('/dr_management')
def dr_management():
    res="select * from doctors"
    db=Db()
    qry=db.select(res)
    return render_template("hospital/dr managment.html",data=qry)

@app.route('/edit_dr_details')
def edit_dr_details():
    return render_template("hospital/edit dr details.html")

@app.route('/edit_profile2')
def edit_profile2():
    return render_template("hospital/edit profile.html")

@app.route('/edit_schedule')
def edit_schedule():
    return render_template("hospital/edit schedule.html")

@app.route('/signup')
def signup():
    return render_template("hospital/signup.html")

@app.route('/signup_post',methods=['post'])
def signup_post():
    name=request.form['textfield']
    place=request.form['textfield2']
    post=request.form['textfield3']
    pin=request.form['textfield4']
    contact=request.form['textfield5']
    email=request.form['textfield6']
    license=request.form['textfield7']
    image=request.files['fileField']
    image.save(static_path + 'hospital\\'+ image.filename)
    path='/static/hospital/' + image.filename
    db=Db()
    qry="INSERT INTO login(`uname`,`password`,`type`)VALUES('"+email+"','"+contact+"','pending')"
    res=db.insert(qry)
    qry1="INSERT INTO hospital(`hlid`,`name`,`place`,`post`,`pin`,`contact`,`email`,`licence`,`image`)VALUES('"+str(res)+"','"+name+"','"+place+"','"+post+"','"+pin+"','"+contact+"','"+email+"','"+license+"','"+path+"')"
    res1=db.insert(qry1)
    return render_template('login.html')



@app.route('/view_booking')
def view_booking():
    res="select * from booking,schedule,user where booking.slid=schedule.sid"
    db=Db()
    qry=db.select(res)
    return render_template("hospital/view booking.html",data=qry)

@app.route('/view_dr_schedule')
def view_dr_schedule():
    res="select * from schedule,doctors where schedule.dlid=doctors.dlid"
    db=Db()
    qry=db.select(res)
    return render_template("hospital/view dr schedule.html",data=qry)

@app.route('/view_drs')
def view_drs():
    res="select * from doctors"
    db=Db()
    qry=db.select(res)
    return render_template("hospital/view drs.html",data=qry)

@app.route('/view_profile3')
def view_profile3():
    return render_template("hospital/view profile.html")




#------------------------------USER

@app.route('/booking_history')
def booking_history():
    res="select * from booking,schedule,doctors"
    db=Db()
    qry=db.select(res)
    return render_template("user/booking history.html",data=qry)

@app.route('/change_password2')
def change_password2():
    return render_template("user/chnage password.html")

@app.route('/complaint')
def complaint():
    return render_template("user/complainnt.html")

@app.route('/edit_profile4')
def edit_profile4():
    return render_template("user/edit profile.html")

@app.route('/image_upload2')
def image_upload2():
    return render_template("user/image upload.html")


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
    print(img)
    path="C:\\Users\\91884\\PycharmProjects\\retinal_disease\\static\\check_img\\"+img.filename
    path1="C:\\Users\\91884\\PycharmProjects\\retinal_disease\\static\\check_img\\a.jpg"

    # CANNY IMAGE
    import cv2
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
    return render_template("/image1.html",)


@app.route('/signup2')
def signup2():
    return render_template("user/Signup.html")

@app.route('/view_doctors')
def view_doctors():
    res="select * from doctors"
    db=Db()
    qry=db.select(res)
    return render_template("user/view doctor.html",data=qry)

@app.route('/view_hospital')
def view_hospital():
    res="select * from hospital"
    db=Db()
    qry=db.select(res)
    return render_template("user/View hospital.html",data=qry)

@app.route('/view_profile4')
def view_profile4():
    return render_template("user/view profile.html")

@app.route('/view_reply')
def view_reply():
    res="select * from complaint"
    db=Db()
    qry=db.select(res)
    return render_template("user/view reply to complaint.html",data=qry)



@app.route('/view_scheduling2')
def view_scheduling2():
    res="select * from schedule"
    db=Db()
    qry=db.select(res)
    return render_template("user/view scheduling.html",data=qry)
if __name__ == '__main__':
    app.run(debug=True,port=1234)
