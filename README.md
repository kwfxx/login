import requests,uuid,random,re,ctypes,json,urllib,hashlib,hmac,urllib.parse,base64,os,string
from time import sleep
import time




timestamp = str(int(time.time()))

def RandomStringUpper(n = 10):
    letters = string.ascii_uppercase + '1234567890'
    return ''.join(random.choice(letters) for i in range(n))
def RandomString(n=10):
    letters = string.ascii_lowercase + '1234567890'
    return ''.join(random.choice(letters) for i in range(n))


def RandomStringUpper(n=10):
    letters = string.ascii_uppercase + '1234567890'
    return ''.join(random.choice(letters) for i in range(n))


def RandomStringChars(n=10):
    letters = string.ascii_lowercase
    return ''.join(random.choice(letters) for i in range(n))


def randomStringWithChar(stringLength=10):
    letters = string.ascii_lowercase + '1234567890'
    result = ''.join(random.choice(letters) for i in range(stringLength - 1))
    return RandomStringChars(1) + result


uu = '83f2000a-4b95-4811-bc8d-0f3539ef07cf'


def generate_DeviceId(ID):
        volatile_ID = "12345"
        m = hashlib.md5()
        m.update(ID.encode('utf-8') + volatile_ID.encode('utf-8'))
        return 'android-' + m.hexdigest()[:16]


class sessting:
    def __init__(self):
        pass
    def headers_login(self):
        headers = {}
        headers['User-Agent'] = self.UserAgent
        headers['Host'] = 'i.instagram.com'
        headers['content-type'] = 'application/x-www-form-urlencoded; charset=UTF-8'
        headers['accept-encoding'] = 'gzip, deflate'
        headers['x-fb-http-engine'] = 'Liger'
        headers['Connection'] = 'close'
        return headers
    def generateUSER_AGENT(self):
        Devices_menu = ['HUAWEI', 'Xiaomi', 'samsung', 'OnePlus']
        DPIs = [
            '480', '320', '640', '515', '120', '160', '240', '800'
        ]
        randResolution = random.randrange(2, 9) * 180
        lowerResolution = randResolution - 180
        DEVICE_SETTINTS = {
            'system': "Android",
            'Host': "Instagram",
            'manufacturer': f'{random.choice(Devices_menu)}',
            'model': f'{random.choice(Devices_menu)}-{randomStringWithChar(4).upper()}',
            'android_version': random.randint(18, 25),
            'android_release': f'{random.randint(1, 7)}.{random.randint(0, 7)}',
            "cpu": f"{RandomStringChars(2)}{random.randrange(1000, 9999)}",
            'resolution': f'{randResolution}x{lowerResolution}',
            'randomL': f"{RandomString(6)}",
            'dpi': f"{random.choice(DPIs)}"
        }
        return '{Host} 155.0.0.37.107 {system} ({android_version}/{android_release}; {dpi}dpi; {resolution}; {manufacturer}; {model}; {cpu}; {randomL}; en_US)'.format(
            **DEVICE_SETTINTS)
    def generate_DeviceId(self , ID):
        volatile_ID = "12345"
        m = hashlib.md5()
        m.update(ID.encode('utf-8') + volatile_ID.encode('utf-8'))
        return 'android-' + m.hexdigest()[:16]
    

class login:
    def __init__(self):
        self.sesstings = sessting()
        self.coo = None
        self.token = None
        self.mid = None
        self.DeviceID = None
        self.sessionid = None
        self.Login()
    
    
    def headers_login(self):
        headers = {}
        headers['User-Agent'] = self.sesstings.generateUSER_AGENT()
        headers['Host'] = 'i.instagram.com'
        headers['content-type'] = 'application/x-www-form-urlencoded; charset=UTF-8'
        headers['accept-encoding'] = 'gzip, deflate'
        headers['x-fb-http-engine'] = 'Liger'
        headers['Connection'] = 'close'
        return headers
        
        
        
        
    def checkpoint(self):
        info = requests.get(f"https://i.instagram.com/api/v1{self.req.json()['challenge']['api_path']}", headers=self.headers_login(), cookies=self.coo)
        step_data = info.json()["step_data"]
        if "phone_number" in step_data:
            try:
                phone = info.json()["step_data"]["phone_number"]
                print(f'[0] phone_number : {phone}')
            except:
                pass
        elif "email" in step_data:
            try:
                email = info.json()["step_data"]["email"]
                print(f'[1] email : {email}')
            except:
                pass

        else:
            print("unknown verification method")
            input()
            exit()
        return self.send_choice()
    def send_choice(self):
        choice = input('choice : ')
        data = {}
        data['choice'] = str(choice)
        data['_uuid'] = uu
        data['_uid'] = uu
        data['_csrftoken'] = 'massing'
        challnge = self.req.json()['challenge']['api_path']
        self.send = requests.post(f"https://i.instagram.com/api/v1{challnge}",headers=self.headers_login(), data=data, cookies=self.coo)
        contact_point = self.send.json()["step_data"]["contact_point"]
        print(f'code sent to : {contact_point}')
        return self.get_code()
    def get_code(self):
        try:
            code = input("code : ")
            data = {}
            data['security_code'] = str(code),
            data['_uuid'] = uu,
            data['_uid'] = uu,
            data['_csrftoken'] = 'massing'
            path = self.req.json()['challenge']['api_path']
            send_code = requests.post(f"https://i.instagram.com/api/v1{path}", headers=self.headers_login(), data=data, cookies=self.coo)
            if "logged_in_user" in send_code.text:
                print(f'Login Successfully as @{self.username}')
                self.coo = send_code.cookies
                self.token = self.coo.get("csrftoken")
                self.mid = self.coo.get("mid")
                self.sessionid = self.coo.get("sessionid")
                print(self.sessionid)
            else:
                regx_error = re.search(r'"message":"(.*?)",', send_code).group(1)
                print(regx_error)
                ask = input("Code is Not Work Do You Want Try Agin [Y/N] : ")
                if ask.lower() == "y":
                    sleep(1)
                    return self.get_code()
                else:
                    exit()
        except:
            print("accepted Done")
            return self.Login()

        
        
    def Login(self):
        self.username = input(f'UserName? : ')
        self.DeviceID = self.sesstings.generate_DeviceId(self.username)
        self.passwordd = input(f'Password? : ')
        data = {}
        data['guid'] = uu
        data['enc_password'] = f"#PWD_INSTAGRAM:0:{timestamp}:{self.passwordd}"
        data['username'] = self.username
        data['device_id'] = self.DeviceID
        data['login_attempt_count'] = '0'

        self.req = requests.post("https://i.instagram.com/api/v1/accounts/login/", headers=self.headers_login(), data=data)
        if "logged_in_user" in self.req.text:
            print(f'Login Successfully as @{self.username}')
            self.coo = self.req.cookies
            self.token = self.coo.get("csrftoken")
            self.mid = self.coo.get("mid")
            self.sessionid = self.coo.get("sessionid")
            print(f"session : {self.sessionid}")
        elif 'checkpoint_challenge_required' in self.req.text:
            self.coo = self.req.cookies
            self.token = self.coo.get("csrftoken")
            self.mid = self.coo.get("mid")
            self.sessionid = self.coo.get("sessionid")
            print("SCURE FOUND ")
            return self.checkpoint()
        else:
            try:
                regx_error = re.search(r'"message":"(.*?)",', self.req.text).group(1)
                print(regx_error)
            except:
                print(self.req.text)
            ask = input("Something has gone wrong Do You Want Try Agin [Y/N] : ")
            if ask.lower() == "y":
                sleep(1)
                os.system("cls")
                return self.login()
            else:
                input()
                exit()

login()
