---
title: Maven plugin that will improve your security

---
Recently we did a lot of security improvements on all our sites it was long needed and we want to have state of the art security on our websites.

As part of our security projects we went over all of our maven dependencies, cross checked them against vulnerability databases and updated most of them. It’s pointless to have secured app if you are running XML parsing library that allows remote code execution.

This process was very time consuming and not easily repeatable. So I started looking for way to automate it, that way we can check all our dependencies with each release, weekly or daily.

I found this awesome [OWASP Dependency Check plugin](https://www.owasp.org/index.php/OWASP_Dependency_Check) that is doing exactly that, it goes over all your dependencies and fails build when some of them new in some vulnerability database. Perfect lets implement that.

Implementation seems easy enough. You just add this XML snippet to your plugin and it should work.

    <plugin>
        <groupId>org.owasp</groupId>
        <artifactId>dependency-check-maven</artifactId>
        <version>3.0.2</version>
        <executions>
            <execution>
                <goals>
                    <goal>check</goal>
                </goals>
            </execution>
        </executions>
    </plugin>

After I have done that my build time was suddenly more than half an hour longer.

Why?

I started investigating that and the plugin downloads all the databases to your local maven repository and does the checks without need to be online. Awesome. So we cannot run this with every build. 30 minutes is a long time and we are doing multiple of those during a day.

We create a special maven profile, if you run build with that profile, the plugin will be used otherwise it will not run.

    <profile>
      <id>owasp</id>
      <build>
        <plugins>
          <plugin>
            <groupId>org.owasp</groupId>

As we now need to run special build for this I created a Jenkins job that will run the build with the profile once a week.

Another issue we ran into is false positives. We have some of our internal libraries that are named similar to some libraries that are in some databases and we were hitting those with every build. Making it fail every week. Some developed needed to go to that job and see if only those are there or something more valid is in the report. Not ideal.

Another option that this plugin allows you to do is a to have a suppression list. You basically add the xml file into your project that is listing stuff that should be ignored. Report even offers you a XML snippet for ignoring that vulnerability. **Do not abuse this, use this only when there is no other option.**

[More on suppression here.](https://jeremylong.github.io/DependencyCheck/general/suppression.html)