# coletar-leitura-de-agua-e-luz-automatico
Com este codigo estou iniciando um projeto, de forma resumida e simples, eu quero poder auxiliar na coleta de leitura de agua e luz nas residencias, o formato atual é 100% humano e acaba havendo dezenas de problemas, tais como: Erro de leitura por não enxergar bem, e não haver a leitura pois o colaborador esta com problemas de saude etc...

1. Código para o Hidrômetro (usando ESP8266 e Wi-Fi):

import machine
import network
import urequests
import time

# Configurações Wi-Fi
SSID = "Seu_SSID"
PASSWORD = "Sua_Senha"

# Configuração do sensor de fluxo de água
sensor = machine.Pin(14, machine.Pin.IN)

# Configuração da conexão Wi-Fi
wifi = network.WLAN(network.STA_IF)
wifi.active(True)
wifi.connect(SSID, PASSWORD)

# Esperar a conexão Wi-Fi
while not wifi.isconnected():
    time.sleep(1)

print("Conectado ao Wi-Fi")

# Função para medir o fluxo de água
def medir_fluxo():
    # Lógica para medir o fluxo de água
    # Por exemplo, contando pulsos do sensor
    pulsos = 0
    for i in range(10):
        if sensor.value() == 1:
            pulsos += 1
        time.sleep(0.1)
    return pulsos

# Função para enviar dados à nuvem
def enviar_dados(consumo):
    url = "https://www.corsan.com/api/hidrometro"
    dados = {"consumo": consumo, "cliente_id": "Divino Clossi"}
    try:
        response = urequests.post(url, json=dados)
        print(response.text)
    except Exception as e:
        print("Erro ao enviar dados:", e)

# Loop principal
while True:
    consumo = medir_fluxo()
    enviar_dados(consumo)
    time.sleep(3600)  # Envia os dados a cada hora
2. Código Backend (Python Flask):

from flask import Flask, request, jsonify
from datetime import datetime
import sqlite3

app = Flask(__name__)

# Conexão com o banco de dados
def connect_db():
    conn = sqlite3.connect('corsan.db')
    return conn

# Rota para receber dados do hidrômetro
@app.route('/api/hidrometro', methods=['POST'])
def receber_dados():
    data = request.get_json()
    consumo = data['consumo']
    cliente_id = data['cliente_id']
    timestamp = datetime.now()

    # Armazenar os dados no banco de dados
    conn = connect_db()
    cursor = conn.cursor()
    cursor.execute("INSERT INTO leituras (cliente_id, consumo, timestamp) VALUES (?, ?, ?)",
                   (cliente_id, consumo, timestamp))
    conn.commit()
    conn.close()

    return jsonify({"status": "sucesso", "cliente_id": cliente_id, "consumo": consumo})

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
