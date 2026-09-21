import cv2
import mysql.connector as ms
import os
import face_recognition
import datetime

#Load Known faces and their names from the 'faces' folder
known_faces=[]
known_names = []

for filename in os.listdir('faces'):
    image = face_recognition.load_image_file(os.path.join('faces', filename))
    encoding = face_recognition.face_encodings(image)[0]
    known_faces.append(encoding)
    known_names.append(os.path.splitext(filename)[0])

vc = cv2.VideoCapture(0+cv2.CAP_DSHOW)

attendance_marked = False

mydb= ms.connect(host='localhost',user='root',passwd='root',database='attendance')
today=datetime.datetime.today().strftime('%d_%m_%Y')
cursor=mydb.cursor()
cursor.execute("Show Tables Like %s",(today,))
result=cursor.fetchone()

if result is None:
    cursor.execute(f"Create table {today} (name Varchar(20),time Varchar(10))")
    mydb.commit()

cursor.close()

while True:
    ret, frame = vc.read()
    rgb_frame = cv2.cvtColor(frame, cv2.COLOR_BGR2RGB)
    face_locations=face_recognition.face_locations(rgb_frame)
    face_encodings=face_recognition.face_encodings(rgb_frame,face_locations)
    recognised_name=[]

    for face_encoding in face_encodings:
        matches=face_recognition.compare_faces(known_faces,face_encoding)
        name='Unknown'

        if True in matches:
            matched_indices=[i for i,match in enumerate(matches) if match]
            for index in matched_indices:
                name=known_names[index]
                recognised_name.append(name)


    if len(recognised_name)>0:
        current_time=datetime.datetime.now().strftime('%H:%M:%S')
        cursor=mydb.cursor()

        for name in recognised_name:
            cursor.execute(f"Select * from {today} where name=%s",(name,))
            result=cursor.fetchone()

            if result is None:
                sql=f"Insert into {today} (name,time) Values(%s,%s)"
                val=(name,current_time)
                cursor.execute(sql,val)
                mydb.commit()

        attendance_marked=True

    cv2.imshow('Camera',frame)

    if cv2.waitKey(50) & 0xFF == ord('q') or attendance_marked:
        break

vc.release()
cv2.destroyAllWindows()
