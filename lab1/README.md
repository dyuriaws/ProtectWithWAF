<!-- TOC -->

-   [Introduction](#introduction)
-   [JAM preperation](#jam-preperation)
-   [Connecting to the Windows Instance](#connecting-to-the-windows-instance)
-   [Step 1: Simulate Cross-Site Scripting, SQL Injection, HTTP Flood, and Bad BOT activities](#step-1-simulate-cross-site-scripting-sql-injection-http-flood-and-bad-bot-activities)
    -   [Simulate a cross site scripting attack.](#simulate-a-cross-site-scripting-attack)
    -   [Simulate a SQL injection attack.](#simulate-a-sql-injection-attack)
    -   [Simulate a HTTP Flood attack](#simulate-a-http-flood-attack)
    -   [Simulate a Bad Bot attack](#simulate-a-bad-bot-attack)
-   [Step 2: Create a WAF and configure protection against the above attacks using WAF native rules.](#step-2-create-a-waf-and-configure-protection-against-the-above-attacks-using-waf-native-rules)
-   [Step 3: Demonstrate protection against the above attacks using WAF native rules.](#step-3-demonstrate-protection-against-the-above-attacks-using-waf-native-rules)
-   [Appendix](#appendix)

<!-- /TOC -->
Lab 1 -- Manually testing attacks and protections
=================================================
### Introduction:
You will be using this Windows instance to simulate web application attacks using the following tools, which have been pre-installed on the Windows Attacker instance, which you will be using throughout lab.
*   Chrome Browser
*   <a href="https://jmeter.apache.org/">Apache JMeter</a>
*   <a href="https://httpd.apache.org/docs/2.4/programs/ab.html">Apache Benchmark (Ab)</a>
*   <a href="https://www.httrack.com/">HTTRack</a>

Then you will be creating an AWS WAF to protect against these simulated attacks and validating the web application is protected by these attacks.

***

### JAM preperation:

1.  It's recommended to open an additional tab in your browser to the <https://jam.awsevents.com> page to be able to get information from the **AWS Account** page and the **Output Properties** page as you proceed in the lab.

2.  Download the SSH Key Pair by selecting **AWS account** on the left side and select the SSH Key Pair **Download** button.
<img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab1/media/jam3.png" width="700" />

3.  In **Output Properties** launch the link for **UpdateAttakerSecurityGroup**. This will determine your internet IP address and add the appropriate security to allow access to resources we will be using in the lab.
<img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab1/media/jam5.png" />
<img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab1/media/jam6.png" width="700" />

4.  While in **Output Properties** note there several other properties we will be using throughout the labs.

***

### Connecting to the Windows Instance:
You will be using Windows Remote Desktop (RDP - Remote Desktop Protocol) to connect to an EC2 windows instance in AWS.  You will need to have a remote desktop client installed to procceed with the lab.  If you are not formiliar with how to connect to an EC2 instance I would suggest reviewing the full instructions on the AWS website <a href="https://docs.aws.amazon.com/AWSEC2/latest/WindowsGuide/connecting_to_windows_instance.html#rdp-prereqs">Connecting to Your Windows Instance</a>.  Here is a summary of the steps you will need to perform to connect to the Attacker instance throughout the lab.

1.  You will need your EC2 keypair pem file (\<keypair filename>.pem), which was used to create these instances.
2.  Login to the **AWS console** and choose the **EC2** service, typing *EC2* in the **Find Services** search is one way to get to the EC2 console.
3.  Be sure to select the appropriate region from the top right drop down box.  This should match where the lab resources are deployed.
4.  From the menu on the left choose **Instances**
5.  Select the **Attacker** instance by selecting the box to the left of the instance name.  This should be the only instance selected.
6.  Near the top of the EC2 console select the **Connect** button
7.  Select the **Get Password** button
8.  Select **Choose File** and supply the keypair pem file
9.  Select **Decrypt Password**
10. Hover your mouse just over to the right of the password displayed and you should see an icon which when clicked will copy the password.
11. Select **Download Remote Desktop File** and then run it.
12. You should be prompted for user credentials and the user name should be prefilled with **Administrator** and you can paste in the password copied above.  Once you have supplied the ID/Password click the appropriate button to proceed.
    *   While connecting you will be prompted about a certificate not being verified. This should occur twice, click continue each time.
    *   If you have the option to save the password feel free to check that box before connecting.
    *   If you are not prompted for credentials it's likely the security group on your instance is not allowing your source IP address.  Please make sure you have completed all of preperation steps.  You can also review your security group rules assigned to the ec2 instance.

***

### Step 1: Simulate Cross-Site Scripting, SQL Injection, HTTP Flood, and Bad BOT activities

#### Simulate a cross site scripting attack.
We will simulate a simple cross site scripting attack by placing some HTML code in the search area, which will embed the alert code into the search results page.  It will then create a browser alert when you click on when you click on any of the links on the search results page.

1.  Remote desktop (RDP) to the **Attacker** windows instance, using the [Preparation](#preparation) steps above.
    *   **Note:** Once you login you may be prompted to allow your computer to be discoverable.  You can select **No**, but either will be fine.

2.  On the Attacker instance (RDP Session) open Chrome browser.  It should open the the webcarter ALB URL by default.  Might be in another tab.  Clicking the **Home** button should get you there as well, but you can also find the url JAM **Output Properties** page:
    <img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab1/media/image2.png" />

3.  In the browser window on the WebCarter site type the following in the **Search** text field and click **Search!**:
    ```html
    <script>alert(document.cookie)</script>
    ```
4.  Click on any of the Brands filters (**Apple**, **NBA**). The script will be executed and the session cookie will be printed in the alert. Your Cross-site scripting attack was successful:
    <img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab1/media/image4.png" />

#### Simulate a SQL injection attack.
We will simulate a SQL injection attack by placing a sql query in the the search area.  This will throw a database error on the page when you execute the search.

5.  In the **Search** field enter the following and click **Search!**:
    ```
    ' and 1=0 union select 1,group_concat(table_name) from information_schema.tables #
    ```
    <img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab1/media/image6.png" />

6.  Server returns **Database Error** page displaying the SQL statement been executed. You successfully performed a SQL Injection attack and got a desired output from the server.
    <img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab1/media/image7.png" />

#### Simulate a HTTP Flood attack
We will simulate a HTTP Flood attack by using JMeter to send lots of traffic to the web page.

7.  Run the DOS scenario Apache JMeter by clicking on the DOS shortcut on the desktop.

    <img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab1/media/image8.png" />

8.  Click the green play arrow in the top menu ribbon to start the scenario:
    <img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab1/media/image11.png" />

9.  Navigate to **View Results Tree**, select one of the items which has one of the item with a green shield under, and assure you're getting 200 OK responses. Your HTTP Flood attack was successful:
    <img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab1/media/image12.png" />

10. Stop and Clear the results tree for the next tests:
    <img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab1/media/image13.png" />

#### Simulate a Bad Bot attack
We will simulate a bad bot attack by using Apache Benchmark to send 100 http requests using the User-Agent = bad-bot.

11. Open a Command window using the **CMD** shortcut on the desktop and change directory to **C:\Apache24\bin**.
    <img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab1/media/image36.png" />

12. Run Apache Benchmark:
    *   Copy the abLoadTestCommand from the **Output Properties** tab on the JAM webpage.  Right click and copy or select the copy icon to the right of the command.

        <img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab1/media/jam7.png" width="700" />
    *   Run the command in a Command Prompt window
    *   Assure that ab run above was successful:
        <img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab1/media/image35.png" />

***

### Step 2: Create a WAF and configure protection against the above attacks using WAF native rules.

1.  Login to AWS console and choose EC2 service.

2.  Choose **Load Balancers** in the left menu and then choose your load balancer created by the lab.

3.  Go to **Integrated Services** tab and click on **Create Web ACL** button. You'll be directed to the **WAF & Shield** service console.

4.  Choose the region you're using in the **Filter** field.  This is defined in the JAM **AWS Account** page, but should be **us-west-2** or **US West (Oregon)**:
    <img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab1/media/image15.png" />
    **Note**: WAF configuration is separate for each region and Global for CloudFront deployments. You must configure your Web ACL in the same region where you have your ALB deployed.

5.  Click **Create web ACL**, fill out the details as below, and click **Next**:

    **Note**: Please select the region the ALB is deployed.
    ><img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab1/media/image16.png" />

6.  Create Cross-site scripting match condition. Click **Add Filter** then **Create**.

    ><img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab1/media/image17.png" />

    **Note**: Filters in the conditions are logically disjoined, i.e. **OR**. You can have up to 10 filters in each condition (hard limit).




7.  Create SQL injection match condition. Click **Add Filter** then **Create**.

    ><img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab1/media/image18.png" />




8.  Create String and regex match condition. Click **Add Filter** then **Create**.

    ><img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab1/media/image19.png" />




9.  Click **Next** on the *Create Conditions* page.

10. Click **Create Rule** button.

    **Note:** Be sure to pick the appropriate match condition for each of the rules you are creating.  For example the **XSS1** condition you would want to select the **match at least one of the filters in the cross-site scripting match condition**.  In the screenshots below each rule include the appropriate match condition.

11. Create a simple Cross-site scripting rule that in our example will contain a single XSS condition created above. Select the condition **XSS1** as below and click **Add Condition**:

    ><img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab1/media/image20.png" />

    **Note**: Conditions in the rule are logically joined, i.e. **AND**. You can have up to 100 conditions of each type per account (soft limit).




12. Create SQL Injection rule choosing the SQL injection condition created above:

    ><img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab1/media/image21.png" />




13. Create Bad Bot rule choosing the String match condition created above:

    ><img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab1/media/image22.png" />




14. Create HTTP Flood rule changing the Rule type to **Rate-based rule**:

    ><img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab1/media/image23.png" />

    **Note**: No conditions are needed for this rule, but you could add match conditions to the rate-based rule when crafting your own custom signatures.




15. Set default Web ACL action as **Allow all requests that don't match any rules** and click **Review and create**.

16. Review the Web ACL and click **Confirm and create**.

**Note**: Because we started creating a Web ACL from the ALB console, your Web ACL is already associated with your load balancer. You also can just create Web ACL and then associate it with one or more load balancers.

***

### Step 3: Demonstrate protection against the above attacks using WAF native rules.

In this step we'll repeat the attacks in Step 1 above and assure they're blocked by the rules we created in Step 2.

1.  In a Chrome browser window navigate to your ALB endpoint and type following in the **Search** string.  Then click **Search!**.
    ```html
    <script>alert(document.cookie)</script>
    ```

2.  Assure you get HTTP 403 response from WAF:
    <img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab1/media/image24.png" />

3.  In the **Search** field enter the following:
    ```
    ' and 1=0 union select 1,group_concat(table_name) from information_schema.tables #
    ```

4.  Assure you get HTTP 403 response from WAF:
    <img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab1/media/image24.png" />

5.  Start JMeter DOS scenario again, clicking the green play arrow in the menu ribbon, and navigate to the Results tree. Watch the request till they start failing after reaching 2000 requests / 5 min as configured in your rate-based rule:
    <img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab1/media/image25.png" />

6.  Go to AWS WAF Console and navigate to **Rules-\> HTTPFlood1Rule**.  Notice IP addresses that are currently blocked by this rule:

    <img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab1/media/image26.png" />

    These IP addresses configured in JMeter in **CSV Data Set Config** section and originate from the attacker instance.

7.  Repeat ab test as above and notice Non-2xx responses from WAF:
    <img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab1/media/image27.png" />

In this lab you protected your Web application against Cross-site scripting, SQL Injection, HTTP flood and simple BOTs using AWS WAF native rules.

***

### Appendix:
*   When opening the AWS WAF & Shield console you maybe prompted with the WAF & Shield splash screen instead WAF specific console.

    <img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab1/media/image28.png" />
*   Click the **Go to AWS WAF** button
    <img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab1/media/image29.png" />
*   Then click **Web ACLs**
*   Update the **Filter** to reflect the appropriate region where the ALB is deployed.

    <img src="https://aws-jam-challenge-resources.s3.amazonaws.com/WAF-BOTs-Scrapers-Workshop/lab1/media/image34.png" />
