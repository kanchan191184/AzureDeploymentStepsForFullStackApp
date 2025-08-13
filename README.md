# AzureDeploymentStepsForFullStackApp
Microsoft Azure Deployment Steps for full stack app with tech stack - Spring boot (backend), MySQL DB, ReactJS (Frontend) using Azure Container App, Storage account and Database for MySQL flexible server.

# Create MIcrosoft Azure Account
# For FrontEnd ReactJS deployment 

  Run this command to build the frontend React app. 
    - Go to frontend codefolder
    - Run this command
      npm run build
    - This will create a dist folder at the root of frontend app.
    - This dist folder contains a folder named assets (having two files, one .js and other .css), one inde.html     file.
1. Go on Azure and search for "Storage account" service. Create a new storage account, choose General-purpose v2 and for redundancy option, choose LRS (Locally Redundant Storage).
2. Go to settings > static website
3. ensure that static website hosting is enabled.
4. Set the Index document name (usually index.html) - this tells Azure what file to serve when the root URL is requested.
5. Optionally, set the Error document path (like 404.html) if applicable

## Update your Frontend API calls
Update your API calls in your React code from relative paths to include the full backend URL.

## Upload files into Storage account

  - When static website hosting is enables, Azure creates a special container named $web where you must upload the files.
  - Upload the index.html file that you have in dist folder in your project root folder.
  - Also upload the assets folder, along with both .js and .css files.
## You can access the website using the URL provided there. 

## Backend Spring boot setup
  You need to create a Dockerfile because Docker won't know how to build or package into an image for your app without it. 
  - Create a "Dockerfile" at the root of project with this code:
     
    FROM openjdk:21-jdk-slim          //(whatever jdk version you are using)
    WORKDIR /app
    COPY target/<Your-jar-file-name>.jar app.jar        //(Your jar file name)
    EXPOSE 8080           //Port number
    ENTRYPOINT ["java", "-jar", "app.jar"]

# Deploy/redeploy docker file
  -run this command to build your spring boot jar:
    mvn clean package
  - Build docker image using this command:
    docker build -t employeecreator:latest .
  - Verify the build :
    docker images
    
    
  

## Create a Azure Database for MySQL flexible servers
1. Open the "Azure Database for MySQL flexible servers" from Azure resources.
2. Click create button to create new one.
3. Choose "quick create" option inside Flexible server.
4. Choose your subscription and Resource group
5. Enter administrator lagin and password for mysql.
6. Rest options keep as it is.

## Connect Azure Database for MySQL Flexible Server with Azure Container app
   1.Define profiles in "application.properties
    # Default profile (local development)
    spring.profiles.active=local
    # Local MySQL configuration
    spring.datasource.url=jdbc:mysql://localhost:3306/your_local_db
    spring.datasource.username=your_local_user
    spring.datasource.password=your_local_password
    spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
    spring.jpa.hibernate.ddl-auto=update
    spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL8Dialect
    # Azure MySQL configuration (will be overridden in application-azure.properties)
    
  2. Create Azure-Specific Configuration
     - Create a file named "application-azure.properties" in the same directory where you have "application.properties" file with this content and replace the placeholders with your actual Azure MySQL server details.
       # Azure MySQL configuration
      spring.datasource.url=jdbc:mysql://<azure_mysql_server_name>.mysql.database.azure.com:3306/your_azure_db?useSSL=true&requireSSL=true
      spring.datasource.username=your_azure_user@<azure_mysql_server_name>
      spring.datasource.password=your_azure_password
      spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
      spring.jpa.hibernate.ddl-auto=update
      spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL8Dialect

  3. Activate the Appropriate Profile
     - For Local development, ensure that
       spring.profiles.active = local
       is set in your application.properties file.
     - For Azure deployment, set the sctive profile to azure
        spring.profiles.active=azure
  4. Secure senstitive Information
    For better security, especially in production environments, avoid hardcoding sensitive information like database passwords in your configuration files. Instead, consider using environment variables or Azure Key Vault to store and retrieve sensitive data securely.


5. Got to your MySQL database server that you have created 
5. Test connectivity using MySQL CLI:
     mysql -h employee-mysql.mysql.database.azure.com -u youruser@employee-mysql -p

## To set enviornment variables you need to go to Container app in Azure.
- From left side, choose Security > Secrets
- In Secrets, Add key and value (Key should be smallcase)
- Then from left side, choose Application > Containers > Environment variables (from tab).
- Add new environment variables name (same as you are using in .env file in your code)
- Choose source as "Reference a secret" and choose value from dropdown. (You can see all the secrets that you have added previously, you can choose one of them.)
-    

## Connect the Frontend to your Spring Boot Backend
1. Create a config file in src/main/java/project/<your-project-name>/ with name as WebConfig.java
2. Put this code in there

    import org.springframework.context.annotation.Bean;
    import org.springframework.web.servlet.config.annotation.CorsRegistry;
    import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;
    
    @Bean
    public WebMvcConfigurer corsConfigurer() {
        return new WebMvcConfigurer() {
            @Override
            public void addCorsMappings(CorsRegistry registry) {
                registry.addMapping("/**")
                        .allowedOrigins("https://<your-frontend-domain>")
                        .allowedMethods("GET", "POST", "PUT", "DELETE");
            }
        };
    }

This setup ensures your frontend can communicate safely with your backend without running into browser security blocks.

3. Use the environment variables when building/running, create 2 files named .env.development and .env.production at the root of frontend code(same level as src folder of frontend )
   - When running locally, .env.development file will with this code
       VITE_API_URL=http://localhost:8080
   - When deploying to Azure Storage static website, use .env.production with your deployed backend URL
      VITE_API_URL=https://<your-backend-URL>

## Build your frontend for production again 
- Run the build command at frontend folder
      npm run build
- Upload the new build files to your Azure storage $web container
