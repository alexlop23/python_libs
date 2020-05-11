import json
import datetime
import smtplib
from O365 import Message
from elasticsearch import Elasticsearch
import smtplib
from email.mime.text import MIMEText
from email.mime.multipart import MIMEMultipart
from email.mime.base import MIMEBase
from email import encoders
import os.path
from flask import Flask,request, jsonify
from flask_restful import Resource, Api, reqparse

app = Flask(__name__)
api = Api(app)
@app.route('/foo', methods=['POST'])




def elk():
    server=Elasticsearch("http://172.17.186.55:9200")

    data = request.json
    print("Estos son los datos")
    uid=json.dumps(data['uid'])
    print(uid)
    time_stamp0=json.dumps(data['entry'])
    time_stamp1=json.dumps(data['response'])



    index_T0= time_stamp0.index('T')
    hora0=time_stamp0[index_T0+1] + time_stamp0[index_T0+2]
    minutos0=time_stamp0[index_T0+4] + time_stamp0[index_T0+5]
    segundos0=time_stamp0[index_T0+7] + time_stamp0[index_T0+8]

    index_T= time_stamp1.index('T')
    hora1=time_stamp1[index_T+1] + time_stamp1[index_T+2]
    minutos1=time_stamp1[index_T+4] + time_stamp1[index_T+5]
    segundos1=time_stamp1[index_T+7] + time_stamp1[index_T+8]
    print("Entry time")
    print(hora0,minutos0,segundos0)
    print("Response time")
    print(hora1,minutos1,segundos1)

    total_hours=int(hora1)-int(hora0)
    total_min=int(minutos1)-int(minutos0)
    total_seg=int(segundos1)-int(segundos0)
    print(total_hours, total_min, total_seg)
    tiempo_total= str(total_hours)+"-"+str(total_min)+"-"+str(total_seg)
    mensaje="Tiempo excedido, uid: "+ uid+"Tiempo (hh-mm-ss): "+ tiempo_total


    print(hora0,minutos0,segundos0)
    print("Response time")
    print(hora1,minutos1,segundos1)

    total_hours=int(hora1)-int(hora0)
    total_min=int(minutos1)-int(minutos0)
    total_seg=int(segundos1)-int(segundos0)
    print(total_hours, total_min, total_seg)
    tiempo_total= str(total_hours)+"-"+str(total_min)+"-"+str(total_seg)



    if total_hours>0 or total_min>0 or total_seg>=20:
        entry_data = server.search(index="bcmonpro", doc_type="_doc", body={"query": {"bool": {"must": [{"match": {"uid": uid}},{"match": {"tipo": "Entry"}}]}}})
        response_data = server.search(index="bcmonpro", doc_type="_doc", body={"query": {"bool": {"must": [{"match": {"uid": uid}},{"match": {"tipo": "Response"}}]}}})
        for doc_0 in entry_data['hits']['hits']:
            print("Entro al for")
            entry_msj=json.dumps(doc_0['_source']['message'])

        for doc in response_data['hits']['hits']:
            response_msj=json.dumps(doc['_source']['message'])
            service_response= json.dumps(doc['_source']['serviceresponse'])
        print(response_msj)
        print("lanza alerta")
        mensaje="Tiempo excedido, uid: "+ uid+ '\n' + "Tiempo (hh-mm-ss): "+ tiempo_total + '\n' + "Mensaje de entrada: " + '\n' + entry_msj + '\n' + "Mensaje de respuesta: " + '\n' + response_msj + '\n' +"Service Response: "+ service_response
        print(mensaje)

        send_email('alopez@palo-it.com','Accion Urgente Requerida',mensaje)

    elif total_seg>=15 and total_seg<20:
        entry_data = server.search(index="bcmonpro", doc_type="_doc", body={"query": {"bool": {"must": [{"match": {"uid": uid}},{"match": {"tipo": "Entry"}}]}}})
        response_data = server.search(index="bcmonpro", doc_type="_doc", body={"query": {"bool": {"must": [{"match": {"uid": uid}},{"match": {"tipo": "Response"}}]}}})
        for doc_0 in entry_data['hits']['hits']:
            print("Entro al for")
            entry_msj=json.dumps(doc_0['_source']['message'])

        for doc in response_data['hits']['hits']:
            response_msj=json.dumps(doc['_source']['message'])
            service_response= json.dumps(doc['_source']['serviceresponse'])
        print(response_msj)
        print("lanza alerta")
        mensaje="Tiempo excedido, uid: "+ uid+ '\n' + "Tiempo (hh-mm-ss): "+ tiempo_total + '\n' + "Mensaje de entrada: " + '\n' + entry_msj + '\n' + "Mensaje de respuesta: " + '\n' + response_msj + '\n' +"Service Response: "+ service_response
        print(mensaje)

        send_email('alopez@palo-it.com','Alerta Time out',mensaje)
    else:
        print("todo en orden")

    return jsonify(data)

def send_email(email_recipient,
               email_subject,
               email_message):

    email_sender = 'monitoreo-produccion@cardif.com.mx'

    msg = MIMEMultipart()
    msg['From'] = email_sender
    msg['To'] = email_recipient
    msg['Subject'] = email_subject

    msg.attach(MIMEText(email_message, 'plain'))

    try:
        server = smtplib.SMTP(10.170.136.26, 25)
        server.ehlo()
        server.starttls()
        server.login('monitoreo-produccion@cardif.com.mx', ' ')
        text = msg.as_string()
        server.sendmail(email_sender, email_recipient, text)
        print('email sent')
        server.quit()
    except:
        print("SMPT server connection error")
    return True


if __name__ == "__main__":
  app.run(debug=True, port=4002)

elk()
