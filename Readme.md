Install sonarqube plugin

add below to pom.xml
<plugin>
                <groupId>org.sonarsource.scanner.maven</groupId>
                <artifactId>sonar-maven-plugin</artifactId>
                <version>3.11.0.3922</version>
            </plugin>






kubectl create secret docker-registry acr-secret \
	--docker-server=democontainerregistry12.azurecr.io \
	--docker-username=democontainerregistry12 \
	--docker-password=tY84N7CWJ0kXLcdIsICK3M85wP7f5JITPyZ99vtT6ikLD2bovVe4JQQJ99CIACYeBjFEqg7NAAACAZCRFnMZ