#!/usr/bin/env python
# CJSubmit.py
# (c) 2014 David J Shuckerow (djs0017@auburn.edu)
#
# Takes functionality originally from CJClient.py
# and enables it as a command-line interface to the server.
#
#

import sys, struct, socket
from string import maketrans

#connection data
address = ""
port = ""
#password data
loginID = 0
loginPW = ""

FILE_ERRORS = {}
FILE_ERRORS[0] = "PROBLEM SOLVED."
FILE_ERRORS[1] = "COMPILATION ERROR."
FILE_ERRORS[2] = "TIMELIMIT ERROR."
FILE_ERRORS[3] = "NO OUTPUT OR RUNTIME ERROR."
FILE_ERRORS[4] = "INCORRECT OUTPUT LENGTH OR RUNTIME ERROR."
FILE_ERRORS[5] = "INCORRECT OUTPUT OR RUNTIME ERROR."

def encrypt(line):
	table = maketrans(
			"abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ1234567890",
			"qcWhICRpbEeyYADoLiNzMrPnlawFkxTKQXmVftvuGZsgSUJdHjBO0987654321")
	return line.translate(table)

def decrypt(line):
	table = maketrans(
			"qcWhICRpbEeyYADoLiNzMrPnlawFkxTKQXmVftvuGZsgSUJdHjBO0987654321",
			"abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ1234567890")
	return line.translate(table)

def readConfig():
	global address, port, loginID, loginPW
	try:
		with open("/usr/local/bin/cjsubmit.conf") as f:
			address = decrypt(f.readline().strip())
			port = int(f.readline().strip())
			loginID = int(f.readline().strip())
			loginPW = decrypt(f.readline().strip())
	except:
		pass

def writeConfig():
	with open("/usr/local/bin/cjsubmit.conf", 'w') as f:
		s = "{}\n{}\n{}\n{}\n".format(
			encrypt(address),port,loginID,encrypt(loginPW))
		f.write(s)
	
class CJSubmitter():
	def __init__(self, serverAddress, serverPort):
		print("Setting up the socket...")
		sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
		sAddress = (serverAddress, serverPort)
		print("Connecting to %s port %s" % sAddress)
		sock.connect(sAddress)
		self.connection = sock
		self.sAddress = sAddress
	def submitFile(self,filename, problem):
		print "Send file {} to solve {}".format(filename,problem)
		self.connection.sendall(bytearray([3]))
		print "Validating Team ID..."
		self.sendln(str(loginID))
		self.sendln(str(loginPW))
		if self.recvbool():
			print "Validation Successful."
		else:
			print "Validation Failed.  Configure UserID and Password with -u idnumber and -p password"
			return
		print "Sending file."
		if (not self.fileExists(filename)):
			self.connection.sendall(bytearray([0]))
			print "File does not exist!  Submission Cancelled.  You will not receive a penalty."
			return
		self.connection.sendall(bytearray([2]))
		self.sendfile(filename)
		self.sendln(problem)
		print "File sent.  Your submission is now running or in a queue to run."
		print "Result will be printed out here when run is completed."
		print "You may work on new code, but wait to submit again until you have received your response."
		errorcode = bytearray(self.connection.recv(1))[0]
		print FILE_ERRORS[errorcode]
		
	def recvbool(self):
		# Get a boolean
		ret = (self.connection.recv(4))
		if ret:
			num = int(struct.unpack("<I", ret)[0])
			return num != 0
		return False
	def recvln(self):
		datalen = ((self.connection.recv(4)))
		s = ''
		if datalen:
			num = int(struct.unpack("<I", datalen)[0])
			print "Receive {} characters".format(num)
			if num > 0:
				s = self.connection.recv(num)
				print s
			print "Received {} characters".format(len(s))
		else:
			return None
		return s
	def recvfile(self):
		# Get a filename
		filename = self.recvln()
		contents = self.recvln()
		return filename, contents
	def sendbool(self, b):
		# Send a boolean to a recipient
		self.connection.sendall(bytearray(struct.pack('<I',b)))
	def sendln(self, line):
		# Send a line of input
		num = len(line)
		self.connection.sendall(bytearray(struct.pack('<I',num)))
		self.connection.sendall(bytearray(line.strip()))
	def sendfile(self, filename):
		contents = ''
		with open(filename) as f:
			contents = ''.join(f.readlines())
		self.sendln(filename.split('/')[-1])
		self.sendln(contents)
		print contents
	def savefile(self,filename,contents):
		with open(filename, 'w') as f:
			f.write(contents)
			f.close()
	def fileExists(self, filename):
		try:
			with open(filename):
				pass
			return True
		except:
			return False


# Evaluate all the arguments.
readConfig()
for i in range(1,len(sys.argv)-1, 2):
	tag = sys.argv[i]
	val = sys.argv[i+1]
	print tag, val
	if tag == '-p':
		loginPW = val.strip()
		writeConfig()
	elif tag == '-u':
		loginID = int(val.strip())
		writeConfig()
	elif tag == '-ip':
		address = val.strip()
		writeConfig()
	elif tag == '-pt':
		port = int(val.strip())
		writeConfig()
	elif tag == '-h' or tag == '-help':
		pass
	else:
		# Submit a file.
		s = CJSubmitter(address, port)
		filename = tag
		problem = val
		s.submitFile(filename, problem)

if len(sys.argv) <= 2:
	print "cjsubmit: Improper usage."
	print "Usage: "
	print "$ cjsubmit [filename problemname] [-u userID] [-p password] [-ip ServerIPAddress] [-pt ServerPort]"
		
