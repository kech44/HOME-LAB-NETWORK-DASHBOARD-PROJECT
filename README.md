# HOME-LAB-NETWORK-DASHBOARD-PROJECT

**ICT171- HOME LAB NETWORK DASHBOARD

NAME: ALLAN KOECH

STUDENT NUMBER: 35043207**


Domain: https://www.birirallan.com/

This project is a lightweight, self‑hosted network monitoring system that scans your LAN and WIFI, measures latency, checks device status, records network speed, and displays everything in a clean web dashboard.
The project consists of:
- A windows-based network scanner (nmap.py)
- A flask backend API (app.py)
- A JSON datastore (data.json)
- A web dashboard (index.html,dashboard.js.style.css)

** FEATURES**
  WIFI device discovery
  latency for each device
  download and upload speed
  Timestamped updates
  clean JSON update

 **Project structure**
1. Web Dashboard
 Note: The server used is Amazon EC2 running Ubuntu with Nginx installed
create your HTML and paste it into index.html after using terminal to remotely connect to your server through SSH using the following command with information from the server

                 ssh -i "yourkeyname.pem" ubuntu@your_ec2_public_dns_or_IP
     Index.html is edited using the command:
   
                  nano /var/www/html/index.html
   
   Create the style.css file and the dashboard.js file and move them to a directory homelab-dashboard. The commands to create the files if they are not created yet;


         sudo nano /var/www/homelab-dashboard/index.html

          sudo nano /var/www/homelab-dashboard/styles.css
   
          sudo nano /var/www/homelab-dashboard/dashboard.js

the following is samples of each index.html, styles.css and dashboard.js

Index.html;

     <!DOCTYPE html>
    <html lang="en">
     <head>
    <meta charset="UTF-8">
      <title>Home Network Dashboard</title>
         <link rel="stylesheet" href="styles.css">
          <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
       </head>
  
     <body>
        <h1>Home Network Monitoring Dashboard</h1>

    <section id="summary">
        <div class="card"><h2>Total Devices</h2><p id="total-devices">0</p></div>
        <div class="card"><h2>Online Devices</h2><p id="online-devices">0</p></div>
        <div class="card"><h2>Average Latency</h2><p id="avg-latency">0 ms</p></div>
        <div class="card"><h2>Last Update</h2><p id="last-update">--</p></div>
    </section>

    <h2>Device List</h2>
    <table id="device-table">
        <thead>
            <tr>
                <th>IP Address</th>
                <th>MAC Address</th>
                <th>Status</th>
            </tr>
        </thead>
        <tbody></tbody>
    </table>

    <h2>Latency Over Time</h2>
    <canvas id="latencyChart"></canvas>

    <script src="dashboard.js"></script>
    </body>
      </html>


Styles.css;

    body {
    font-family: Arial, sans-serif;
    background: #f4f4f4;
    padding: 20px;
    }

     h1, h2 {
    text-align: center;
     }

    #summary {
     display: flex;
    justify-content: space-around;
    margin-bottom: 30px;
      }

     .card {
    background: white;
    padding: 20px;
    border-radius: 8px;
    width: 22%;
    text-align: center;
    box-shadow: 0 2px 5px rgba(0,0,0,0.1);
    }

    table {
    width: 100%;
    border-collapse: collapse;
    background: white;
    margin-bottom: 30px;
    }
  
    table th, table td {
    padding: 12px;
    border: 1px solid #ddd;
    }

    .online {
    color: green;
    font-weight: bold;
     }

    .offline {
    color: red;
    font-weight: bold;
    }
 dashboard.js;

     async function fetchData() {
    const response = await fetch("/api/devices");
    const data = await response.json();

    // Summary
    document.getElementById("total-devices").textContent = data.devices.length;
    document.getElementById("online-devices").textContent =
        data.devices.filter(d => d.online).length;
    document.getElementById("avg-latency").textContent = data.latency_ms + " ms";
    document.getElementById("last-update").textContent = data.timestamp;

    // Device table
    const tbody = document.querySelector("#device-table tbody");
    tbody.innerHTML = "";

    data.devices.forEach(device => {
        const row = document.createElement("tr");
        row.innerHTML = `
            <td>${device.ip}</td>
            <td>${device.mac}</td>
            <td class="${device.online ? 'online' : 'offline'}">
                ${device.online ? "Online" : "Offline"}
            </td>
        `;
        tbody.appendChild(row);
    });

    // Update chart
      addLatencyPoint(data.latency_ms);
    }

    let latencyData = [];
    let chart;

    function setupChart() {
    const ctx = document.getElementById("latencyChart").getContext("2d");
    chart = new Chart(ctx, {
        type: "line",
        data: {
            labels: [],
            datasets: [{
                label: "Latency (ms)",
                data: latencyData,
                borderColor: "blue",
                fill: false
            }]
        }
    });
    }

    function addLatencyPoint(latency) {
    const now = new Date().toLocaleTimeString();
    chart.data.labels.push(now);
    chart.data.datasets[0].data.push(latency);
    chart.update();
    }

    setupChart();
    fetchData();
    setInterval(fetchData, 10000);


  The dashboard layout looks as follows;


<img width="1820" height="946" alt="image" src="https://github.com/user-attachments/assets/4860b95d-3edf-489f-911d-6689b1bb238d" />


  
   The command to create the directory to put the index.html, styles.css and dashboard.js is;

       sudo mkdir -p /var/www/homelab-dashboard

   The command to move the files to the directory is;

       sudo cp index.html styles.css dashboard.js /var/www/homelab-dashboard/

   Next is to create a Nginx server block to host the website
   The following command is used to create and edit the server block;

       sudo nano /etc/nginx/sites-available/homelab-dashboard

  The format should be similar to;
 

           server {
 
                  listen 80;
       server_name example.com www.example.com;

        root /var/www/://example.com;
        index index.html index.htm;

        location / {
        try_files $uri $uri/ =404;
          } 
        }

The following command is to enable the Nginx site;

    sudo ln -s /etc/nginx/sites-available/homelab-dashboard /etc/nginx/sites-enabled/

The following command should reload nginx;

       sudo systemctl reload nginx

       
The dashboard should now be live at your IP address or domain name.

To enable HTTPS(SSL/TLS) to your domain the following command can be used;

    sudo apt install certbot python3-certbot-nginx -y
    sudo certbot --nginx -d your.domain.name

    
  Your domain should now be accessible through;
  
         https://your.domain.name


2. Build the Backend API to give the dashboard real data

-It accepts POST data from the home monitoring script
-It saves it to a JSON file
-It serves it the the dashboard
In this project Python Flask was used as it was the easiest to use.


Fisrt Flask is installed on the EC2 server;
     
     sudo apt update
     sudo apt install python3 python3-pip -y
     pip3 install flask

     
The API directory is created;

    sudo mkdir -p /opt/homelab-api
    cd /opt/homelab-api

    
Create app.py which is a tiny Flask server to read the data.json file to make it available to the dashboard at /api/devices

      sudo nano /opt/homelab-api/app.py

The following is a sample python code of app.py;


    from flask import Flask, request, jsonify
    import json
    import os
    from datetime import datetime

     app = Flask(__name__)

     DATA_FILE = "data.json"

    # Load existing data or create empty structure
     def load_data():
    if os.path.exists(DATA_FILE):
        with open(DATA_FILE, "r") as f:
            return json.load(f)
    return {"timestamp": None, "latency_ms": 0, "devices": []}

    def save_data(data):
    with open(DATA_FILE, "w") as f:
        json.dump(data, f, indent=4)

    @app.route("/api/update", methods=["POST"])
    def update():
    data = request.json
    if not data:
        return jsonify({"error": "Invalid JSON"}), 400

    data["timestamp"] = datetime.utcnow().isoformat()
    save_data(data)

    return jsonify({"status": "ok"}), 200

     @app.route("/api/devices", methods=["GET"])
    def devices():
    return jsonify(load_data())

    if __name__ == "__main__":
    app.run(host="127.0.0.1", port=5000)

The API can be run using the following command after creating it;

    python3 /opt/homelab-api/app.py
    
Create the data.json file on the linux server that stores the data;
go into the API  directory;

    cd /opt/homelab-api

create the file;

    sudo nano data.json


paste the starter template;

     {
    "timestamp": "",
     "download_mbps": 0,
     "upload_mbps": 0,
     "devices": []
    }




3.Python Home monitoring script

This script will run on the windows machine collecting network information
Install the required tools which are;
-Python
-Nmap for scanning the network
- Python script for sending the data to the server

i) Install python on windows

 Download Python from:

    https://www.python.org/downloads/windows/
    
ii) Download and install Nmap to use on Windows to collect the data

     https://nmap.org/download.html

iii)Write the python script to initiate and send data to the server;

    import subprocess
    import re
    import requests

    API_URL = "https://your.domain.name/api/update"

    def scan_network():
    result = subprocess.check_output("nmap -sn 192.168.1.0/24", shell=True).decode()

    devices = []
    current_ip = None

    for line in result.split("\n"):
        if "Nmap scan report for" in line:
            current_ip = line.split(" ")[-1]
        if "MAC Address" in line:
            mac = line.split(" ")[2]
            devices.append({"ip": current_ip, "mac": mac, "online": True})

    return devices

    def main():
    devices = scan_network()
    payload = {"latency_ms": 0, "devices": devices}
    requests.post(API_URL, json=payload)

    if __name__ == "__main__":
    main()
Save the python file to the windows machine and run it in powershell to send data to the server the output should look as follows;

<img width="822" height="942" alt="image" src="https://github.com/user-attachments/assets/6a74e139-9aa8-435f-8f46-9b56aa207ec6" />



  
