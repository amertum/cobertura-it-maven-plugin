# cobertura-it-maven-plugin

Fork of codehaus [cobertura-maven-plugin](http://mojo.codehaus.org/cobertura-maven-plugin) which enable the so long waited integration test coverage feature [MCOBERTURA-86](http://jira.codehaus.org/browse/MCOBERTURA-86).

# How to use

## Required

See [How to do integration test with maven-failsafe-plugin](http://maven.apache.org/plugins/maven-failsafe-plugin/usage.html)

## Add pluginsRepositories configuration to your POM

    <pluginRepositories>
        <pluginRepository>
            <id>cobertura-it-maven-plugin-maven2-release</id>
            <url>http://cobertura-it-maven-plugin.googlecode.com/svn/maven2/releases</url>
            <snapshots>
                <enabled>false</enabled>
            </snapshots>
            <releases>
                <enabled>true</enabled>
            </releases>
        </pluginRepository>
    </pluginRepositories>

## Repository URLs

  * release : http://cobertura-it-maven-plugin.googlecode.com/svn/maven2/releases

### Add plugin configuration to your POM

This plugin provides two more goals :

  * check-only : check cobertura coverage **without** invoking the execution of the lifecycle phase test prior to executing itself.
  * report-only : report cobertura coverage **without** invoking the execution of the lifecycle phase test prior to executing itself.

## Integration tests coverage for a single module JAR/WAR

This is the simplest usage of this plugin. It allows running cobertura:check-only during the maven verify phase.

Run it with : <code>mvn clean verify site -Pit-coverage</code>

    <profile>
        <id>it-coverage</id>
        
        <build>
            <plugins>
                <plugin>
                    <groupId>org.codehaus.mojo</groupId>
                    <artifactId>cobertura-it-maven-plugin</artifactId>
                    <version>2.5</version>
                    <configuration>
                        <formats>
                            <format>xml</format>
                        </formats> 
                        <check>
                            <haltOnFailure>false</haltOnFailure>
                        </check>
                    </configuration>
                    <executions>
                        <execution>
                            <id>cobertura-clean</id>
                            <phase>clean</phase>
                            <goals>
                                <goal>clean</goal>
                            </goals>
                        </execution>
                        <execution>
                            <id>cobertura-instrument</id>
                            <phase>process-classes</phase>
                            <goals>
                                <goal>instrument</goal>
                            </goals>
                        </execution>
                        <execution>
                            <id>cobertura-check-only</id>
                            <phase>verify</phase>
                            <goals>
                                <goal>check-only</goal>
                            </goals>
                        </execution>
                    </executions>
                </plugin>
            </plugins>
        </build>
        
        <reporting>
            <plugins>
                <plugin>
                    <groupId>org.codehaus.mojo</groupId>
                    <artifactId>cobertura-it-maven-plugin</artifactId>
                    <configuration>
                        <formats>
                            <format>html</format>
                            <format>xml</format>
                        </formats>
                    </configuration>
                    <reportSets>
                        <reportSet>
                            <reports>
                                <report>report-only</report>
                            </reports>
                        </reportSet>
                    </reportSets>
                </plugin>
            </plugins>
        </reporting>
    </profile>

## Fonctional tests coverage for a multi-module WAR application

To enabled coverage for a WAR application within a multi-module project where you want the  coverage of the dependencies modules, the instrumented classes should be packaged within the WAR and the dependencies as well.
So the trick to do it, is to ignore the dependencies during WAR packaging and unpack the dependencies sources within the WAR so they will be instrumented. Then the WAR should be deploy (using lightweight container such as Jetty) and tests run.

    <profile>
        <id>it-coverage</id>
        
        <build>
            <plugins>
                <plugin>
                    <artifactId>maven-dependency-plugin</artifactId>
                    <executions>
                        <execution>
                            <id>unpack-dependencies-src</id>
                            <phase>generate-sources</phase>
                            <goals>
                                <goal>unpack-dependencies</goal>
                            </goals>
                            <configuration>
                                <classifier>sources</classifier>
                                <includeGroupIds>com.example</includeGroupIds>
                                 <outputDirectory>${project.build.directory}/generated-sources/it-dependencies</outputDirectory>
                            </configuration>
                        </execution>
                    </executions>
                </plugin>
                <plugin>
                    <groupId>org.codehaus.mojo</groupId>
                    <artifactId>build-helper-maven-plugin</artifactId>
                    <executions>
                        <execution>
                            <id>add-it-dep-source</id>
                            <phase>generate-sources</phase>
                            <goals>
                                <goal>add-source</goal>
                            </goals>
                            <configuration>
                                <sources>
                                    <source>${project.build.directory}/generated-sources/it-dependencies</source>
                                </sources>
                            </configuration>
                        </execution>
                    </executions>
                </plugin>
    
                <plugin>
                    <groupId>org.codehaus.mojo</groupId>
                    <artifactId>cobertura-it-maven-plugin</artifactId>
                    <version>2.5</version>
                    <configuration>
                        <formats>
                            <format>xml</format>
                        </formats> 
                        <check>
                            <haltOnFailure>false</haltOnFailure>
                        </check>
                    </configuration>
                    <executions>
                        <execution>
                            <id>cobertura-clean</id>
                            <phase>clean</phase>
                            <goals>
                                <goal>clean</goal>
                            </goals>
                        </execution>
                        <execution>
                            <id>cobertura-instrument</id>
                            <phase>process-classes</phase>
                            <goals>
                                <goal>instrument</goal>
                            </goals>
                        </execution>
                        <execution>
                            <id>cobertura-check-only</id>
                            <phase>verify</phase>
                            <goals>
                                <goal>check-only</goal>
                            </goals>
                        </execution>
                    </executions>
                </plugin>
                
                <plugin>
                    <artifactId>maven-war-plugin</artifactId>
                    <configuration>
                        <archiveClasses>false</archiveClasses>
                        <packagingExcludes>WEB-INF/lib/example-*.jar</packagingExcludes>
                    </configuration>
                </plugin>
            </plugins>
        </build>
        
        <reporting>
            <plugins>
                <plugin>
                    <groupId>org.codehaus.mojo</groupId>
                    <artifactId>cobertura-it-maven-plugin</artifactId>
                    <version>2.5</version>
                    <configuration>
                        <formats>
                            <format>html</format>
                            <format>xml</format>
                        </formats>
                    </configuration>
                    <reportSets>
                        <reportSet>
                            <reports>
                                <report>report-only</report>
                            </reports>
                        </reportSet>
                    </reportSets>
                </plugin>
            </plugins>
        </reporting>
    </profile>
