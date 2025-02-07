<p align="center">
<img src="https://i.imgur.com/Ua7udoS.png" alt="Traffic Examination"/>
</p>

<h1>Network Security Groups (NSGs) and Inspecting Traffic Between Azure Virtual Machines</h1>
In this tutorial, we observe various network traffic to and from Azure Virtual Machines with Wireshark as well as experiment with Network Security Groups. <br />

<h2>Environments and Technologies Used</h2>

- Microsoft Azure (Virtual Machines/Compute)
- Remote Desktop
- Various Command-Line Tools
- Various Network Protocols (SSH, RDP, DNS, HTTP/S, ICMP)
- Wireshark (Protocol Analyzer)

<h2>Operating Systems Used </h2>

- Windows 10 (21H2)
- Ubuntu Server 20.04

<h2>Pre-Reqs </h2>

- 2 VMs on the same virtual network (one running Linux, one running windows)
- Wireshark installed on the Windows machine

<h2>Actions and Observations</h2>

![Screenshot 2025-02-07 073841](https://github.com/user-attachments/assets/d5d530c7-f989-4882-ad42-a91b890ee6c4)


<p>
  First, I will connect to the window using RDP on my end and open Wireshark within the windows machine. The first window that will pop up will show you some different options, we are going to choose 'Ethernet' and click the shark icon at the top left to start capturing the packets of data being sent to and from our machine.
</p>

---

![Screenshot 2025-02-07 074255](https://github.com/user-attachments/assets/3ef5d7e4-97ab-4658-95ff-4f0548e0ceb5)


<p>
  You should be presented with a spam of information. In the top bar where it says 'Apply a display filter ...', we are going to type 'icmp' this will filter out all the data and only show us packets related to the ICMP protocol (what the ping command uses). You should be happy to see the spam now stopped because of the filter we placed.
</p>

---

![Screenshot 2025-02-07 080727](https://github.com/user-attachments/assets/f197ea80-9e34-4e17-ab79-241593923e9c)
![Screenshot 2025-02-07 080653](https://github.com/user-attachments/assets/ee078d5a-d84d-447f-b985-deb601b11fef)



<p>
  Next, we are going to find the private IP address of the linux machine (a VM in azure for me), if you created a VM in azure for this, you can find this info inside Virtual Machines > (your linux VM name) > Overview > Private IP address. 
  
  Inside of the windows VM, I am going to open powershell and type 'ping (Linux VM's Private IP address). Now, you will ONLY see the packets of data that were sent and recieved from the ping request that we made, since it uses the icmp protocol that we filtered for.
</p>

---

![Screenshot 2025-02-07 082615](https://github.com/user-attachments/assets/07d882b1-84ef-4b23-b617-2baa6840dfa6)


<p>
  Now, we are going to type the same ping command but add '-t' to the end of it, this will let the ping command execute infinitely. As expected, you will see an infinite spam on both the powershell and packets being captured by Wireshark. 
</p>

---

![Screenshot 2025-02-07 082908](https://github.com/user-attachments/assets/56c98da9-59c6-4be6-8ecf-42654853750b)
![Screenshot 2025-02-07 083422](https://github.com/user-attachments/assets/acfd2315-25cc-429b-9761-cf51ee303867)


<p>
  Back in azure, we are going to head to the linux VM, and in the dropdowns tab to the left head to 'Networking' > 'Network settings' and we will click on the Network security group (NSG). Then here, we are going to head to 'Settings' > 'Inbound security rules' and hit 'Add' up top. We will set the 'Destination port ranges' to * (meaning any port), and select 'ICMPv4' and set the 'Priority' to a number lower than any of the standard ones, I will set it to '290' this makes it the first rule followed. After clicking add, ICMP traffic to the linux VM will be blocked by it.
</p>


---
![Screenshot 2025-02-07 083652](https://github.com/user-attachments/assets/8a6f391b-e14d-42b0-a847-4f9503eb19f0)


<p>
  The rule will takie some time to go in effect, however, eventually, you will see that the request of ping starts timing out, as shown in powershell, and you will see that in Wireshark where we were getting requests and replies back, it is now only showing the request packets being sent since the linux VM is now blocking ICMP traffic. Delete the rule, and observe the ICMP replies eventually coming back!
</p>

---

![Screenshot 2025-02-07 084610](https://github.com/user-attachments/assets/93bb5083-839b-4ade-af2f-8cf39ebd9bf5)


<p>
  Now, we are going to set the filter to 'ssh' in Wireshark and SSH into the linux machine, to do this type 'ssh (your linux username)@(linux machines private IP address)' if you created the linux machine in azure, you will have set this username there. Once you start ineracting with the linux machine via SSH you will notice SSH traffic start popping up in Wireshark, interestly enough, every keystroke sends data! Now, you can simply type 'exit' to end the SSH connection.
</p>

---

![Screenshot 2025-02-07 090128](https://github.com/user-attachments/assets/bee38f48-8011-43d5-9ee6-d4ca25e3f0a7)


<p>
  Now, we are going to observe the assigning of IP addresses using DHCP. Set your filter in Wireshark to 'udp.port == 67 || udp.port == 68'. I am going to create a text file containing 'ipconfig /release ipconfig /renew' and save it to the directory that powershell is working from (C:\Users\(username)) and save it as a .bat file. Then in powershell I am going to type '.\(filename).bat'. This will those commands in powershell. This only needs to be done if you're following along on a VM, because the connection is being dropped.
</p>


---

![Screenshot 2025-02-07 090341](https://github.com/user-attachments/assets/8252ef61-f242-4ee1-8ad5-f202473098db)


<p>
  This will result in the standard DHCP cycle of the machine getting a private IP address from the DHCP server.
</p>


---

![Screenshot 2025-02-07 090559](https://github.com/user-attachments/assets/0dc52803-3ede-4f2f-ba1f-941cd7f2e55d)


<p>
  Now, let's set the filter in Wireshark to 'dns', we will type into powershell 'nslookup google.com'. This will result in quite a bit of DNS traffic packets in Wireshark.
</p>

---

![Screenshot 2025-02-07 090728](https://github.com/user-attachments/assets/39188765-8010-489a-aa4a-b352dfefbb3c)

<p>
  Finally, we will filter for 'rdp'. If you're connected to the VM using remote desktop protocol (RDP), you will see a spam of traffic since we are connected via RDP.
</p>
