

node('master') {

    // download the latest commerce suite from a maven repository
    // and stash the file so it can be copied to each slave 
    // get a pom file and update it to the latest version
    
    stage('Download the commerce suite zip') {
    
        new File("pom.xml") << '''
			<project>
			  <modelVersion>4.0.0</modelVersion>
			  <groupId>com.sap.dbcomp</groupId>
			  <artifactId>jenkinsplay</artifactId>
			  <version>1</version>
			  <dependencies>
				<dependency>
				  <groupId>de.hybris.platform.suite</groupId>
				  <artifactId>commerce-suite</artifactId>
				  <version>1905.0.0.0</version>
				  <type>zip</type>
				</dependency>
			</dependencies>
			</project>
			'''
        sh "mvn versions:use-latest-releases"
        
        sh "mvn dependency:copy-dependencies -DoutputDirectory=."
      
        stash name: 'commerceSuiteZip',
              includes: '*.zip'
    }
}

// The list of database keys for DaaS
List<DatabaseDef> databaseDefs = [
        new DatabaseDef("oracle12c"),
        new DatabaseDef("sqlserver2017")]

// The branches structure is a map from branch name to branch code.
// Given to the 'parallel' build step later:
def branches = [:]

// Loop through the database definitions and define the parallel stages:
for (databaseDef in databaseDefs) {

    String daasKey = databaseDef.daasKey

    String branchName = "Build and test database: " + daasKey
    String outFileName = "results_" + daasKey

    branches[branchName] = {

        // Start the branch with a node definition. We explicitly exclude the
        // master node, so only the two slaves will do builds:
        node('!master') {
			  stage(branchName) {
				try {
					unstash name: 'commerceSuiteZip'

					unzip *.zip

					sh '''
						
curl -X POST -s \\ -F 'format=properties' https://daas.hybris.com/api/db/${bamboo.db.type} >> hybris/config/local.properties" 
echo "init.admin.password=${bamboo.admin.password}" >> hybris/config/local.properties
echo "Local Properties"
cat hybris/config/local.properties
ant clean all
ant initialize
'''
				}
				catch (ignored) {
					currentBuild.result = 'UNSTABLE'
				}
			}
		}
    }
}

parallel branches

// After completing the parallel builds, run the final step on the master node
// that collects the stashed results
node('master') {
    stage('Publish Results') {
    }
}

// Database defintion - currently simple string with DaaS key but could get more sophisticated
class DatabaseDef implements Serializable {

    String daasKey

    StageDef(final String daasKey) {
        this.daasKey = daasKey
    }
}