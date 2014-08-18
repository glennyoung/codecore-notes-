# Pow & Stuff: 
[pow](http://pow.cx/) | [ngrok](https://ngrok.com/)    
Get pow: `curl get.pow.cx | sh`  

To set up a Rack app, just symlink it into `~/.pow`:
```
$ cd ~/.pow
$ ln -s /path/to/myapp
```
Don't forget to [read the manual](http://pow.cx/manual.html).  
  
Restart the server and check the development log
```
touch tmp/restart.txt
tail -f log/development.log
```
If you want to edit the length of the kill time, you can open up `~/.powconfig` and add the following options
```
# ~/.powconfig
export ENABLE_HTTPS="yes"
export POW_TIMEOUT=3600
```