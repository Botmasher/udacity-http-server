# Requests & Responses

## 1. Introduction
- http is lang web browsers and web servers use to speak to each other
- lesson 1: explore building blocks of http
- lesson 2: write web server and client programs from the ground up and handle html form user input
- lesson 3: learn about web server hosting, cookies and practical aspects
- bridge between scripting in Python and understanding of http
- NOT about installing Apache on a Linux server
	- it's about the low-lvl behaviors underlying high-lvl frameworks
	- it's about the protocol itself
- Requirements: Bash, Python3, Git, `ncat` from Nmap network testing toolkit
	- `brew install nmap`
	- check `ncat` works
		- open two terminals and run `ncat -l 9999` (listening on 9999) in one and `ncat localhost 9999` in other
		- text entered into one should show in the other on press return
		- one of the programs is acting as simple network server, the other as client
	- so far this is TCP not yet HTTP

## 2. Your First Web Server
- 