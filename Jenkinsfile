#!groovy

def flags = new BranchFlags("${env.BRANCH_NAME}")

def context = [
    now                 : new Date(),
    branchName          : flags._branchName,
    version             : env.BUILD_NUMBER,
    application         : "coe_ssa_saas",
    applicationUriRoot  : "coe_ssa_saas",
    applicationVersion  : "0.1.0",        
]

println "Pipeline Version='${context.version}'"
println "envName='${ENV_NAME}'"
println "Branch name='${env.BRANCH_NAME}'"
println "Job name='${env.JOB_NAME}'"
println "Build number='${env.BUILD_NUMBER}'"
println "uuid='${context.uuid}'"

try {

    lock("${context.application}-${context.branchName}-build") {

		node("master"){

            try {

                stage("Checkout") {
                    
                    cleanWs()
                    checkout scm
                    
                    def inputData = readFile('Jenkinsfile.UnsecuredSettings.json')
                    context.settings = parseJson(inputData)

                }

                context.settings.ProductIds.Default.each{ productId ->

                    stage("Build Code - ${productId}") {
                        
                        if ( ! context.settings.InfraFlag.Default ){
                            buildCode(productId, context.settings.Environments.Development)
                        }
                                                    
                    }

                    stage("Build Infra - ${productId}") {
                        
                        parallel(
                            mainAPI: { buildInfra(productId, context.settings.Environments.Development) }
                        )                                             

                    }

                }

            }

            finally {

                cleanWs notFailBuild: true

            }
				
		}

    }

}

catch (Exception e){

    println "ERROR: '${context.application}-api' branch '${env.BRANCH_NAME}' build #${env.BUILD_NUMBER} failed with error: ${e}. (<${env.BUILD_URL}|Open>)" 
    throw e

}

@NonCPS
parseJson(inputData) {
    def jsonSlurper = new groovy.json.JsonSlurperClassic()
    def jsonData = jsonSlurper.parseText(inputData)

    return jsonData
}

@NonCPS
def getSetting(settings, settingName, envName) {
    def settingNameCap = settingName.capitalize()

    if (settings[settingNameCap] != null) {
        def envSettings = settings[settingNameCap]
        def envNameCap = envName.capitalize()

        if (envSettings[envNameCap] != null) {
            envSettings[envNameCap]
        } else if (envSettings["Default"] != null) {
            return envSettings["Default"]
        } else {
            return null
        }
    }
}

def echo(String message) {
    bat "@echo ${message}"
}

def buildInfra(def productId, def envName){
    println "Building Infra for ${productId} in ${envName}"
    runTerragrunt(productId,envName)
}

def buildCode(def productId, def envName){
    println "Building Code for ${productId} in ${envName}"

    switch (envName) {
        case "dev":
            runUnitTests(productId,envName)
        case "stage":
            runTerragrunt(productId,envName)
            runUnitTests(productId,envName)
            runIntegrationTests(productId,envName)
            runSecurityTests(productId,envName)
            runLoadTests(productId,envName)
            break;
        case "prod":
            runTerragrunt(productId,envName)
            runLoadTests(productId,envName)
            runSmokeTests(productId,envName)
            break;
        default:
            echo("Do nothing")
    }
}

def runTerragrunt(def productId, def envName){
    def jobName = "test-infra"
    println "Running Terragrunt ${jobName}/${envName}"
    build job: "${jobName}/${envName}", propagate: true, wait: true
}

def triggerBuild(def jobName, def productId, def envName){
    println "Triggering build ${jobName}/${envName}"
    //build job: "${jobName}/${envName}", propagate: true, wait: true
}

def runUnitTests(def productId, def envName){
    println "Running Unit Tests ${productId}/${envName}"
}

def runIntegrationTests(def productId, def envName){
    println "Running Integration Tests ${productId}/${envName}"
}

def runSecurityTests(def productId, def envName){
    println "Running Security Tests ${productId}/${envName}"
}

def runLoadTests(def productId, def envName){
    println "Running Load Tests ${productId}/${envName}"
}

def runSomkeTests(def productId, def envName){
    println "Running Smoke Tests ${productId}/${envName}"
}

class BranchFlags implements Serializable {
    private String _branchName

    BranchFlags(branch) {
        this._branchName = branch.replaceAll('/', '-')
    }

    boolean isMasterBranch() {
        this._branchName == "master"
    }

    boolean isHotfix() {
        this._branchName.startsWith('hf')
    }

    boolean isFeature() {
        this._branchName.startsWith('fb-') || this._branchName.startsWith('feature')
    }

    boolean isDevelopBranch() {
        this._branchName.contains('develop')
    }

    boolean isReleaseBranch() {
        this._branchName.contains('release')
    }

    boolean isHotfixOrFeature() {
        this.isHotfix() || this.isFeature()
    }

    boolean isReleasableBranch() {
        this.isHotfix() || this.isReleaseBranch()
    }
}