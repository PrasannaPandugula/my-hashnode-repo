---
title: "Why your EC2 app isn't opening in Browser (How to Fix It)"
datePublished: Fri Aug 22 2025 13:45:27 GMT+0000 (Coordinated Universal Time)
cuid: cmemvuvyd000402jxd0khajv9
slug: why-your-ec2-app-isnt-opening-in-browser-how-to-fix-it
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1760373665368/26a3ab3d-d587-44da-a5b5-7828f969111c.jpeg
tags: ec2-security-group-inbound-rules

---

When deploying application on AWS EC2, it’s not enough to just run the app inside instance. You must configure the application to listen on the correct network interface (0.0.0.0) and update EC2 security Groups rules to allow inbound traffic on the required ports. without these steps, the app will run internally but won’t be accessible from the browser.

Please follow the steps below to ensure your EC2 app is accessible in the browser.

**1\. Launch EC2 instance** → Choose OS (Ubuntu/Amazon Linux).

**2.Connect via SSH** → ssh -i key.pem ec2-user@&lt;public-ip&gt;

**3.Install dependencies** → Python, Java, Node.

**4.Deploy your app** → clone repo / copy files.

**5.Run app on 0.0.0.0** (not just localhost)

Ex: python [app.py](http://app.py) --host=0.0.0.0 --port=5000

**6.Security Group inbound rules** → open required ports (80/443 for web, 5000 if testing Flask).

AWS console → search EC2 → Instances tab → Security → Security groups → click + Edit inbound rules → add application port.

**7\. Test in browser** → http://:&lt;EC2-public-ip&gt;:&lt;port&gt;

By following these simple steps, you can make sure your application on EC2 is accessible from any browser without issues.

#ec2 security group inbound rules #AWS #EC2 #DevOps #CloudComputing #Deployment.