# Quick Installation #

To install the plugin download the latest jar and pom to a directory and run the following command line :-
```
 mvn install:install-file -Dfile=maven-utplsql-plugin-1.31.jar -DpomFile=maven-utplsql-plugin-1.31-pom.xml
```

# Sample utplsql Test POM #

To quickly get going save the sample pom to your maven structure, patch in your groupId and artifactId, add your test scripts then your off.

NB/
The following test pom features a number of env properties which are typically defined in a maven users profile.

```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<parent>
		<groupId>com.mytest.group</groupId>
		<artifactId>myparentartifact</artifactId>
		<version>1.0.0-SNAPSHOT</version>
	</parent>

	<groupId>om.mytest.group.myparentartifact</groupId>
	<artifactId>my-utplsql-tests</artifactId>

	<name>Sample Maven utplsq Test Setup</name>

	<build>
		<plugins>
			<plugin>
				<groupId>org.codehaus.mojo</groupId>
				<artifactId>sql-maven-plugin</artifactId>
				<version>1.3</version>

				<dependencies>
					<dependency>
						<groupId>com.oracle</groupId>
						<artifactId>ojdbc</artifactId>
						<version>${oracle.version}</version>
					</dependency>
				</dependencies>
				
				<configuration>
					<driver>oracle.jdbc.driver.OracleDriver</driver>
					<url>jdbc:oracle:thin:@${db.host}:1521:${db.instance}</url>
					<username>${db.username}</username>
					<password>${db.password}</password>
					<delimiter>/</delimiter>
					<delimiterType>row</delimiterType>
					<keepFormat>true</keepFormat>
				</configuration>

				<executions>
					<execution>
						<id>install-plsql-test-packages</id>
						<phase>process-test-resources</phase>
						<goals>
							<goal>execute</goal>
						</goals>
						<configuration>
							<orderFile>ascending</orderFile>
							<fileset>
								<basedir>${basedir}/src/test/plsql</basedir>
								<includes>
									<include>**/*.h</include>
									<include>**/*.sql</include>
								</includes>
							</fileset>
						</configuration>
					</execution>
				</executions>
			</plugin>

			<plugin>
				<groupId>com.theserverlabs.maven.utplsql</groupId>
				<artifactId>maven-utplsql-plugin</artifactId>
				<version>1.31-SNAPSHOT</version>

				<dependencies>
					<dependency>
						<groupId>com.oracle</groupId>
						<artifactId>ojdbc</artifactId>
						<version>${oracle.version}</version>
					</dependency>
				</dependencies>

				<configuration>
					<driver>oracle.jdbc.driver.OracleDriver</driver>
					<url>jdbc:oracle:thin:@${db.host}:1521:${db.instance}</url>
					<username>${db.username}</username>
					<password>${db.password}</password>
					<packages>
						<param>pkg1</param>
						<param>pkg2</param>
						<param>pkg3</param>
					</packages>
				</configuration>

				<executions>
					<execution>
						<id>run-plsql-test-packages</id>
						<phase>test</phase>
						<goals>
							<goal>execute</goal>
						</goals>
					</execution>
				</executions>
			</plugin>
		</plugins>
	</build>
</project>
```

# Adding JDBC Driver #

The oracle drivers are not stored in the maven main repositry so if you need to add one use the command line below to add it  :-

```
mvn install:install-file -Dfile=ojdbc14.jar -DgroupId=com.oracle -DartifactId=ojdbc14 -Dversion=10.2.0.3.0 -Dpackaging=jar -DgeneratePom=true
```

# Using Maven to install the Oracle schema, packages etc. #

It is helpful to follow the following project structure when using the plugin. This is roughly based on the Maven defaults:

  * /src/main/plsql/ - PL/SQL source code (packages, functions)
  * /src/test/plsql/ -  PL/SQL unit tests (written in utPLSQL)
  * /src/main/sql/ -  SQL scripts for creating the schema
  * /src/main/resources/data/ -  SQL scripts for inserting schema data

This means that you can use the Maven SQL plugin to install the database schema, PL/SQL packages, data and PL/SQL test packages in the specified Oracle database when you run the maven test command.

The database connection details are specified in the main `<configuration>` element - change these to suit your environment. Although the configuration initially looks quite complicated, it really is just the same thing repeated various times and is easy to understand once you get used to it.

```
      <plugin>
        <groupId>org.codehaus.mojo</groupId>
        <artifactId>sql-maven-plugin</artifactId>
        <version>1.3</version>

        <dependencies>
            <dependency>
                <groupId>com.oracle</groupId>
                <artifactId>ojdbc14</artifactId>
                <version>${oracle.version}</version>
            </dependency>
        </dependencies>

        <configuration>
          <driver>oracle.jdbc.driver.OracleDriver</driver>
          <url>jdbc:oracle:thin:@localhost:1521:xe</url>
          <username>testing</username>
          <password>testing</password>
          <delimiter>/</delimiter>
          <delimiterType>row</delimiterType>
          <keepFormat>true</keepFormat>
        </configuration>

        <executions>
          <execution>
            <id>create-schema</id>
            <phase>process-source-resources</phase>
            <goals>
              <goal>execute</goal>
            </goals>
            <configuration>
              <orderFile>ascending</orderFile>
              <delimiter>;</delimiter>
              <delimiterType>normal</delimiterType>
              <onError>continue</onError>
                <keepFormat>false</keepFormat>
              <fileset>
                <basedir>src/main/sql</basedir>
                  <includes>
                    <include>**/*.sql</include>
                  </includes>
              </fileset>
            </configuration>
          </execution>

          <execution>
            <id>create-plsql-packages</id>
            <phase>process-source-resources</phase>
            <goals>
              <goal>execute</goal>
            </goals>
            <configuration>
              <orderFile>ascending</orderFile>
              <fileset>
                <basedir>src/main/plsql</basedir>
                  <includes>
                    <include>**/*.pks</include>
                    <include>**/*.pkb</include>
                    <include>**/*.sf</include>
                  </includes>
              </fileset>
            </configuration>
          </execution>

          <execution>
            <id>insert-data</id>
            <phase>process-test-resources</phase>
            <goals>
              <goal>execute</goal>
            </goals>
            <configuration>
              <orderFile>ascending</orderFile>
              <delimiter>;</delimiter>
              <delimiterType>normal</delimiterType>
              <onError>continue</onError>
                <keepFormat>false</keepFormat>
              <fileset>
                <basedir>src/main/resources/data</basedir>
                  <includes>
                    <include>**/*.sql</include>
                  </includes>
              </fileset>
            </configuration>
          </execution>

          <execution>
            <id>create-plsql-test-packages</id>
            <phase>process-test-resources</phase>
            <goals>
              <goal>execute</goal>
            </goals>
            <configuration>
              <orderFile>ascending</orderFile>
              <fileset>
                <basedir>src/test/plsql</basedir>
                  <includes>
                    <include>**/*.pks</include>
                    <include>**/*.pkb</include>
                    <include>**/*.sf</include>
                    <include>**/*.sp</include>
                  </includes>
              </fileset>
            </configuration>
          </execution> 

       </executions>
      </plugin>
```