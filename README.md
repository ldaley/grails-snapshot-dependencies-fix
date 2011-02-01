This project gives you a way to enable proper management of SNAPSHOT dependencies coming from Maven repositories with version of Grails earlier than 1.4.

## Instructions

### Build the JAR

This is a Gradle project that builds a JAR file that you need to put in the `lib` directory of your Grails project.

To build the JAR simply run…

    ./gradlew jar

You then find the JAR in `build/libs` called `grails-snapshot-dependencies-fix-0.1.jar`.

### Install JAR in Grails project

You then need to put this JAR in the `lib` directory of each project that you wish to enable SNAPSHOT handling for.

### Configure a snapshot aware resolver

You now need to configure a custom resolver in the `BuildConfig.groovy` file. This is a bit tricky due to the fact that this file is parsed very early during the Grails bootstrap process so we need to add our new JAR to the class path ourselves.

Here's an example:

    grails.project.dependency.resolution = {

        inherits("global")
        log "warn"
        
        repositories {
            // Normal Grails repos
            grailsPlugins()
            grailsHome()
            grailsCentral()

            // Register the new JAR
    		def classLoader = getClass().classLoader
    		classLoader.addURL(new File(baseFile, "lib/grails-snapshot-dependencies-fix-0.1.jar").toURL())
    		
    		// Get a hold of the class for the new resolver
    		def snapshotResolverClass = classLoader.loadClass("grails.resolver.SnapshotAwareM2Resolver")
    		
    		// Define a new resolver that is snapshot aware
    		resolver(snapshotResolverClass.newInstance(name: "spock-snapshots", root: "http://m2repo.spockframework.org/snapshots"))
        }
         
        dependencies {
            // dependencies
        }
        
        plugins {
            // plugins
        }
    }
    
### Configure what changes

Unfortunately this isn't enough to get it working. You also need to tell Grails which dependencies are capable of changing. You can either do this on a dependency by dependency basis by setting the `changing` flag when declaring dependencies…

    grails.project.dependency.resolution = {
        
        inherits("global")
        log "warn"
        
        repositories {
            // Normal Grails repos
            grailsPlugins()
            grailsHome()
            grailsCentral()

            // Register the new JAR
    		def classLoader = getClass().classLoader
    		classLoader.addURL(new File(baseFile, "lib/grails-snapshot-dependencies-fix-0.1.jar").toURL())
    		
    		// Get a hold of the class for the new resolver
    		def snapshotResolverClass = classLoader.loadClass("grails.resolver.SnapshotAwareM2Resolver")
    		
    		// Define a new resolver that is snapshot aware
    		resolver(snapshotResolverClass.newInstance(name: "spock-snapshots", root: "http://m2repo.spockframework.org/snapshots"))
        }
        
        dependencies {
            compile("myorg:myproject:1.0-SNAPSHOT") {
                changing = true
            }
        }
    }

An alternative is to specify a pattern to match against version numbers to determine what changes, which is usually more convenient…

    grails.project.dependency.resolution = {
        
        // Everything with a version that ends with -SNAPSHOT is changing
        chainResolver.changingPattern = ".*-SNAPSHOT"
        
        inherits("global")
        log "warn"
        
        repositories {
            // Normal Grails repos
            grailsPlugins()
            grailsHome()
            grailsCentral()

            // Register the new JAR
    		def classLoader = getClass().classLoader
    		classLoader.addURL(new File(baseFile, "lib/grails-snapshot-dependencies-fix-0.1.jar").toURL())
    		
    		// Get a hold of the class for the new resolver
    		def snapshotResolverClass = classLoader.loadClass("grails.resolver.SnapshotAwareM2Resolver")
    		
    		// Define a new resolver that is snapshot aware
    		resolver(snapshotResolverClass.newInstance(name: "spock-snapshots", root: "http://m2repo.spockframework.org/snapshots"))
        }
        
        dependencies {
            compile("myorg:myproject:1.0-SNAPSHOT")
        }
    }

### About this issue for Grails 1.4

Enabling this functionality will be easier in Grails 1.4.

You can track the issue for this @ http://jira.codehaus.org/browse/GRAILS-7203 