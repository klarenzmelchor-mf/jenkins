#!groovy

def infra = new InfraModel()
def flags = new BranchFlags("${env.BRANCH_NAME}")

def context = [
    now                 : new Date(),
    branchName          : flags._branchName,
    version             : env.BUILD_NUMBER,
    application         : "coe_ssa_saas",
    applicationUriRoot  : "coe_ssa_saas",
    applicationVersion  : "0.1.0",
    profile             : flags.isReleasableBranch() ? "coe_ssa_saas-prod" : "coe_ssa_saas-dev",
    uuid                : flags.isHotfixOrFeature() ? getBranchUuid(flags._branchName) : "",    
    region              : flags.isReleasableBranch() ? "us-east-1" : "us-east-1",
    account             : flags.isReleasableBranch() ? "prod" : "dev",
    accountnumber       : flags.isReleasableBranch() ? "621270530972" : "621270530972",
    isUnique            : flags.isHotfixOrFeature() ? "yes" : "no",
    errorPolicy         : flags.isReleasableBranch() ? "Never" : "Always",
    loggingLevel        : flags.isReleasableBranch() ? "Info" : "Trace",
    s3JobName           : "${env.JOB_NAME}".replaceAll('%2F', '%252F'),        
]

def regionMap = loadRegionMap(flags)

println "Pipeline Version='${context.version}'"
println "Environment='${ENV_NAME}'"
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

                context.settings.ProductIds.Default.each{ prodId ->

                    stage("Build Infra - ${prodId}") {
                        
                        buildInfra(prodId, ENV_NAME)                        

                    }

                    stage("Build Code - ${prodId}") {
                        
                        buildCode(prodId, ENV_NAME)                        

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
def getSetting(settings, settingName, environment) {
    def settingNameCap = settingName.capitalize()

    if (settings[settingNameCap] != null) {
        def envSettings = settings[settingNameCap]
        def environmentCap = environment.capitalize()

        if (envSettings[environmentCap] != null) {
            envSettings[environmentCap]
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

String getBranchUuid(String branchName) { 
    def index = branchName.indexOf('-')
    return index >= 0 ? branchName.substring(index + 1) : branchName
}

def buildInfra(def prodId, def environment){
    println "Building Infra for ${prodId} in ${environment}"

    

}

def buildCode(def prodId, def environment){
    //println "Building Infra for ${prodId} in ${environment}"

    switch (environment) {
        case "dev":
            println "Building Infra for ${prodId} in ${environment}"
        case "stage":
            println "Building Infra for ${prodId} in ${environment}"
            break;
        case "prod":
            println "Building Infra for ${prodId} in ${environment}"
            break;
        default:
            println "Building Infra for ${prodId} in ${environment}"
}

// def triggerBuild(def prodId){
//     println "Triggering build for ${prodId}"

//     switch (prodId) {
//             case "aa" || "AA":
//                 runUnitTest(prodId, environment)
//             case "stage":
                
//                 break;
//             case "prod":
//                 this.stage = "${environment}v1"
//                 this.basePath = 'v1'
//                 this.dbStack = "rds-${context.application}-aurora-${environment}"  //assuming aurora postgres for database???
//                 break;
//             default:
// }

def loadRegionMap(flags){
    return [
        "us-east-1": [
            name             : "US East (N. Virginia)",
            regionCountryCode: "us"
        ],
        "us-east-2": [
            name             : "US East (Ohio)",
            regionCountryCode: "us"
        ],
        "us-west-1": [
            name             : "US West (N. California)",
            regionCountryCode: "us"
        ],
        "us-west-2": [
            name             : "US West (Oregon)",
            regionCountryCode: "us"
        ],
        "ca-central-1": [
            name             : "Canada (Central)",
            regionCountryCode: "ca"
        ],
        "eu-west-1": [
            name             : "EU (Ireland)"
        ],
        "eu-central-1": [
            name             : "EU (Frankfurt)"
        ],
        "eu-west-2": [
            name             : "EU (London)"
        ],
        "ap-northeast-1": [
            name             : "Asia Pacific (Tokyo)"
        ],
        "ap-northeast-2": [
            name             : "Asia Pacific (Seoul)"
        ],
        "ap-southeast-1": [
            name             : "Asia Pacific (Singapore)"
        ],
        "ap-southeast-2": [
            name             : "Asia Pacific (Sydney)"
        ],
        "ap-south-1": [
            name             : "Asia Pacific (Mumbai)"
        ],
        "sa-east-1": [
            name             : "South America (S�o Paulo)"
        ]
    ]
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

class InfraModel implements Serializable {

    String environment
    String properEnvironment
    String stage
    String basePath
    String dbStack
    String stackName
    String dynamoStackName
    String stackEnvironment
    String branchEnvironment

    def reset(context, environment, properEnvironment) {
        this.environment = environment
        this.properEnvironment = properEnvironment
        this.dbStack = "rds-api-shared-aurora-${environment}"
        this.branchEnvironment = environment

        switch (environment) {
            case "dev":
            case "stage":
                this.stage = "${environment}v1"
                this.basePath = 'v1'
                break;
            case "prod":
                this.stage = "${environment}v1"
                this.basePath = 'v1'
                this.dbStack = "rds-${context.application}-aurora-${environment}"  //assuming aurora postgres for database???
                break;
            default:
                this.stage = this.cleanBranchName(context)
                this.basePath = this.cleanBranchName(context)
                break;
        }

        this.stackName = "${context.application}-${environment}"
        this.dynamoStackName = "dynamo-${context.application}-${environment}"
        this.stackEnvironment = "${environment}"
        if (context.uuid != "") {
            this.stackName = "${context.application}-${environment}-${context.uuid}"
            this.stackEnvironment = "${environment}-${context.uuid}"
            this.branchEnvironment = context.uuid
        }
    }

    def cleanBranchName(context) {
        def output

        if (context.branchName.contains("-")) {
            output = context.branchName.replaceAll('-', '')
        }

        if (context.branchName.contains("/")) {
            output = context.branchName.replaceAll('/', '_')
        }

        if (context.branchName.contains('develop')) {
            output = "dev"
        }

        return output
    }
}
