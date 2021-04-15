import RPi.GPIO as GPIO
import os
import socket
from http.server import BaseHTTPRequestHandler, HTTPServer
server = socket.socket() 
server.bind(("192.168.0.139", 5000))
server.listen(4) 
client_socket, client_address = server.accept()
print(client_address, "has connected")
while 1==1:
    recvieved_data = client_socket.recv(1024)
    print(recvieved_data)
def setupGPIO():
    GPIO.setmode(GPIO.BCM)
    GPIO.setwarnings(False)

    GPIO.setup(26, GPIO.OUT)

def setupGPIO():
    GPIO.setmode(GPIO.BCM)
    GPIO.setwarnings(False)

    GPIO.setup(20, GPIO.OUT)
    
def setupGPIO():
    GPIO.setmode(GPIO.BCM)
    GPIO.setwarnings(False)

    GPIO.setup(21, GPIO.OUT)

def getTemperature():
    temp = os.popen("/opt/vc/bin/vcgencmd measure_temp").read()
    return temp


class MyServer(BaseHTTPRequestHandler):

    def do_HEAD(self):
        self.send_response(200)
        self.send_header('Content-type', 'text/html')
        self.end_headers()

    def _redirect(self, path):
        self.send_response(303)
        self.send_header('Content-type', 'text/html')
        self.send_header('Location', path)
        self.end_headers()

    def do_GET(self):
        html = '''
           <html>
           <body 
            style="width:960px; margin: 20px auto;">
           <h1>Welcome to my Raspberry Pi</h1>
           <p>Current GPU temperature is {}</p>
           <form action="/" method="POST">
               <div style="margin-bottom: 20px">
           Turn Relay1:
           <input type="submit" name="submit" value="On">
           <input type="submit" name="submit" value="Off">
       </div>
       <div style="margin-bottom: 20px">
           Turn Relay2:
           <input type="submit" name="sumbit" value="On">
           <input type="submit" name="sumbit" value="Off">
       </div>
       <div style="margin-bottom: 20px">
           Turn Relay3:
           <input type="submit" name="sumbit" value="On">
           <input type="submit" name="sumbit" value="Off">
       </div>
           </form>
           </body>
           </html>
        '''
        temp = getTemperature()
        self.do_HEAD()
        self.wfile.write(html.format(temp[5:]).encode("utf-8"))

    def do_POST(self):

        content_length = int(self.headers['Content-Length'])
        post_data = self.rfile.read(content_length).decode("utf-8")
        post_data = post_data.split("=")[1]
  
        setupGPIO()

        if post_data == 'On':
            GPIO.output(26, GPIO.LOW)
        else:
            GPIO.output(26, GPIO.HIGH)

        print("Relay1 is {}".format(post_data))
        self._redirect('/') # Redirect back to the root url
        
  
  
        if post_data == 'On':
            GPIO.output(20, GPIO.LOW)
        else:
            GPIO.output(20, GPIO.HIGH)

        print("Relat2 is {}".format(post_data))
        self._redirect('/')
    
   
         
        if post_data == 'On':
            GPIO.output(21, GPIO.LOW)
        else:
            GPIO.output(21, GPIO.HIGH)

        print("Relay3 is {}".format(post_data))
        self._redirect('/')

# # # # # Main # # # # #

if __name__ == '__main__':
    http_server = HTTPServer((host_name, host_port), MyServer)
    print("Server Starts - %s:%s" % (host_name, host_port))

    try:
        http_server.serve_forever()
    except KeyboardInterrupt:
        http_server.server_close()
