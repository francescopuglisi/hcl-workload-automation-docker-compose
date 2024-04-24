# HCL Workload Automation

## Introduction
Workload Automation is a complete, modern solution for batch and real-time workload management. It enables organizations to gain complete visibility and control over attended or unattended workloads. From a single point of control, it supports multiple platforms and provides advanced integration with enterprise applications including ERP, Business Analytics, File Transfer, Big Data, and Cloud applications.

Docker adoption ensures standardization of your workload scheduling environment and provides an easy method to replicate environments quickly in development, build, test, and production environments, speeding up the time it takes to get from build to production significantly. Install your environment using Docker to improve scalability, portability, and efficiency.

This readme file contains the high-level steps to deploy all of the Workload Automation product components. However, for more detailed information about configuring a specific component, see:

* [Server](readmes/readme_SERVER.md)
* [Console](readmes/readme_CONSOLE.md)
* [Dynamic Agent](readmes/readme_DYNAMIC_AGENT.md)
* [Zcentric Agent](readmes/readme_ZCENTRIC_AGENT.md)
* [FileProxy](fileproxy/README.md)

## Accessing the container images

You can access the HCL Workload Automation container images from the Entitled Registry:

1. Contact your HCL sales representative for the login details required to access the HCL Entitled Registry.

2. Execute the following command to log in into the HCL Entitled Registry:
      
        docker login -u <your_username> -p <your_entitled_key> hclcr.io

 The images are as follows:

* hclcr.io/wa/hcl-workload-automation-agent-dynamic:10.2.2.00.20240424
* hclcr.io/wa/hcl-workload-automation-server:10.2.2.00.20240424
* hclcr.io/wa/hcl-workload-automation-console:10.2.2.00.20240424

## Other supported tags
* 10.2.0.01.20231201
* 10.2.0.00.20230728
* 10.1.0.04.20231201
* 10.1.0.03.20230511-amd64
* 10.1.0.02.20230301
* 10.1.0.01.20221130
* 10.1.0.00.20220722
* 10.1.0.00.20220512
* 10.1.0.00.20220304
* 9.5.0.06.20230324
* 9.5.0.06.20221216
* 9.5.0.06.20220617
* 9.5.0.05.20211217

## Getting Started

If you want to start the containers via Docker Compose, use the following command to clone the current repository:

    git clone https://github.com/WorkloadAutomation/hcl-workload-automation-docker-compose.git

If you do not have git installed in your environment, download the ZIP file from the main page of the repository:

    Click on "Code" and select "Download ZIP"

If you want to customize the installation parameters, modify the **docker-compose.yml** file.

Accept the product licenses by setting the **LICENSE** parameter to **"accept"** in the **wa.env** file located in the container package as follows: **LICENSE=accept**
	

In the directory where  the **docker-compose.yml** file is located, you can start the containers by running the following command:

    docker-compose up -d

To verify that the containers are started, run the following command:

    docker ps 
    
You can optionally check the container logs using the following command:

    docker-compose logs -f <container_name>
    
Where, <container_name> represents one of the following: wa-server, wa-console or wa-agent.     

### Notes

If your server component uses a timezone different from the default timezone, then to avoid problems with the FINAL job stream, you must update MAKEPLAN within the **DOCOMMAND**, specifying the **timezone** parameter and value. For example, if you are using the America/Los Angeles timezone, then it must be specified as follows:

    $JOBS

    WA_WA-SERVER_XA#MAKEPLAN
    DOCOMMAND "TODAY_DATE=`${UNISONHOME}/bin/datecalc today pic YYYYMMDD`; ${UNISONHOME}/MakePlan -to `${UNISONHOME}/bin/datecalc ${TODAY_DATE}070
    0 + 1 day + 2 hours pic MM/DD/YYYY^HHTT` timezone America/Los_Angeles"
    STREAMLOGON wauser
    DESCRIPTION "Added by composer."
    TASKTYPE OTHER
    SUCCOUTPUTCOND CONDSUCC "(RC=0) OR (RC=4)"
    RECOVERY STOP

## Custom certificates enhancements

Starting from version 10.1, Fix Pack 3, the default SSL certificates are no longer provided with the containers of the server, console, and agent components. To ensure communication between components, you have two different options while performing a fresh installation:

 - **Scenario 1**: Install in each container its own custom certificates, which must be generated by the user (recommended approach).
 - **Scenario 2**: Do not install custom certificates, and use the custom certificate generated by each component during installation.

### Scenario 1 - Generating your own Custom Certificates

To generate the custom certificates on a Windows or UNIX workstation, run the following command:

    openssl genrsa -out ./ca.key 40xx96
    openssl req -x5x09 -new -nodes -key ./ca.key -subj "/CN=WA_ROOT_CA" -days 3650 -out ./ca.crt -config <openssl_dir>/openssl.cnf
    openssl genrsa -des3 -passout pass:<SSL_PASSWORD> -out ./tls.key 40xx96
    openssl req -new -key ./tls.key -passin pass:<SSL_PASSWORD> -out ./tls.csr -config <openssl_dir>/openssl.cnf -subj "/C=<C>/ST=<ST>/L=<L>/O=Global Security/OU=<OU> Department/CN=<COMMON_NAME>" 
    openssl x509 -req -in ./tls.csr -days 3650 -CA ./ca.crt -CAkey ./ca.key -CAcreateserial -out ./tls.crt

After you run the above commands, the following files are created:

 - ca
 - ca.key
 - ca.srl
 - tls
 - tls.csr
 - tls.key

Store these files into a dedicated folder, which you will mount as a volume of your containers.

Perform the following changes in the  **docker-compose.yaml** file for each component:

 - Specify the password of the Trust Store where the certificates are stored in your component in the **SSL_PASSWORD** parameter in the environment section.
 - Specify the path to the custom certificates on your workstation in the volumes section.

### Scenario 2 - Generating certificates at installation time
In this scenario, you only need to set the **SSL_PASSWORD** parameter in each component using the docker-file.yaml.
Set the **SSL_PASSWORD** parameter in the environement section. Each component will then generate its own custom certificate at installation time. 
Remember to run the following commands in the server and console containers to enable communication:

**server**

    ${WA_DIR}/TWS/JavaExt/jre/jre/bin/keytool -importcert -keystore
    ${WA_DIR}/usr/servers/engineServer/resources/security/TWSServerTrustFile.jks -storepass <SSL_PASSWORD> -storetype jks -file <ca_console_file> -alias wa_ca_console -noprompt

**console**

    ${WA_DIR}/TWS/JavaExt/jre/jre/bin/keytool -importcert -keystore ${WAUI_DIR}/java/jre/bin/keytool -importcert -keystore
    {WAUI_DIR}/usr/servers/dwcServer/resources/security/TWSServerTrustFile.jks -storepass <SSL_PASSWORD> -storetype jks -file <ca_server_file> -alias wa_ca_server -noprompt


These commands import the custom certificates of the components into the corresponding truststores. This is required because you have to retrieve manually the generated certificates **(<ca_server_file >** or **<ca_console_file>**), and copy them from one component to another, and then execute the above commands. For example, to import the <ca_console_file> generated in the console container into the server truststore, perform the following steps:

 1. Retrieve the <ca_console_file> from the container.
 2. Store the <ca_console_file> on the server container, either using a shared volume or the**docker cp** command.
 3. Run the command listed above for the server component.

### Upgrade scenario

When upgrading, perform the steps documented above either in scenario 1 or in scenario 2, then set the **SSL_KEY_FOLDER** variable in the environment section of all the components. The value of this variable is the path to the existing folder on each component containing the default certificates coming from the previous release:


    SSL_KEY_FOLDER=<path_to_folder_containing_your_default_certificates>

### Hybrid scenario
You might want to deploy the server component on the cloud, and the console component on-premises. You then need to import the custom certificates that are available in the component in the cloud into the component that is deployed on-premises. 
For example, if the console is on-premises and the server is deployed in a container, you need to import the server certificates into the console truststore, as follows:

    ${WAUI_DIR}/java/jre/bin/keytool -importcert -keystore ${WAUI_DIR}/usr/servers/dwcServer/resources/security/TWSServerTrustFile.jks -storepass <SSL_PASSWORD> -storetype jks -file <ca_server_file> -alias wa_ca_server -noprompt

## Supported Docker versions
This image is officially supported on Docker version 19.x or later.

Support for versions earlier than 19.x is provided on a best-effort basis.

Please see the [Docker installation documentation](https://docs.docker.com/engine/installation/) for details on how to upgrade your Docker daemon. 

## Limitations
The owner of all product files is the wauser user, thus the product does not run as root, but as wauser only. Don not perform the login as root to start processes or execute other commands such as Jnextplan, otherwise it might create some issues.

Limited to amd64 and Linux on Z platforms.

## Additional Information
For additional information on about the HCL Workload Automation, see the [online](https://help.hcltechsw.com/workloadautomation/v1021/index.html) documentation. For technical issues, search for Workload Scheduler or Workload Automation on [StackOverflow](http://stackoverflow.com/search?q=workload+scheduler).

## License
The Dockerfile and associated scripts are licensed under the [Apache License 2.0](http://www.apache.org/licenses/LICENSE-2.0). HCL Workload Automation is licensed under the HCL  License Agreement. This license for HCL Workload Automation can be found [online](). Note that this license does not permit further distribution.

