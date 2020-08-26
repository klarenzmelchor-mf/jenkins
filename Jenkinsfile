def infra = new InfraModel()
def flags = new BranchFlags("${env.BRANCH_NAME}")

library 'common-libraries'

def context = [
    now                 : new Date(),
    branchName          : flags._branchName,
    application         : "coe_ssa_saas",
    applicationUriRoot  : "coe_ssa_saas",
    applicationVersion  : "0.1.0",
    version             : env.BUILD_NUMBER,    
    certificatesBucket  : flags.isReleasableBranch() ? "coe_ssa_saas-certificates" : "coe_ssa_saas-certificates-dev",
    certSuffix          : 'IAM-SAAS',
    domain              : 'us.IAM-SAAS.com',
    domainCountryCode   : 'us',
    authDomain          : 'coe_ssa_saas.auth0.com',
    profile             : flags.isReleasableBranch() ? "coe_ssa_saas-prod" : "coe_ssa_saas-dev",
    uuid                : flags.isHotfixOrFeature() ? getBranchUuid(flags._branchName) : "",    
    region              : flags.isReleasableBranch() ? "us-east-1" : "us-east-1",
    account             : flags.isReleasableBranch() ? "prod" : "dev",
    accountnumber       : flags.isReleasableBranch() ? "621270530972" : "621270530972",
    authorizerName      : flags.isReleasableBranch() ? "some-aa-url" : "some-aa-url",
    isUnique            : flags.isHotfixOrFeature() ? "yes" : "no",
    errorPolicy         : flags.isReleasableBranch() ? "Never" : "Always",
    sslFlags            : flags.isReleasableBranch() ? "Ssl" : "None",
    autoScaleMinSize    : flags.isReleasableBranch() ? 2 : 1,
    autoScaleMaxSize    : flags.isReleasableBranch() ? 2 : 1,
    loggingLevel        : flags.isReleasableBranch() ? "Info" : "Trace",
    s3JobName           : "${env.JOB_NAME}".replaceAll('%2F', '%252F'),        
]

def regionMap = loadRegionMap(flags)

println "Pipeline Version='${context.version}'"
println "Environment='${env}'"
println "Branch name='${env.BRANCH_NAME}'"
println "Job name='${env.JOB_NAME}'"
println "S3 Job name='${context.s3JobName}'"
println "Build number='${env.BUILD_NUMBER}'"
println "uuid='${context.uuid}'"

properties([
        buildDiscarder(logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: '', numToKeepStr: '5')),
        [$class: 'RebuildSettings', autoRebuild: false, rebuildDisabled: false],
        pipelineTriggers([[$class: 'PeriodicFolderTrigger', interval: '1m']])
])

try {
    lock("${context.application}-${context.branchName}-build") {
		node("IAM"){
				try {
				stage("SCM"){
                    WsCleanup()
					checkout scm

                    def inputData = readFile('Jenkinsfile.UnsecuredSettings.json')
                    context.settings = parseJson(inputData)
                    sh '''
                        echo "Stage: SCM"
                    '''
                    
				}

				stage("Build") {
                    //setupbuild([workingDir: "src", codePath: "iam.microservice1.dockerfile", params: ""])  //params could be ports output to etc
                    //artifactsbuild([workingDir: "src", codePath: "iam.microservice1.dockerfile", params: ""])
                    sh '''
                        echo "Stage: Build"
                    '''
				}

                stage('Package') {
                    steps{
                        packageIAM()
                   }
                }
				stage("Unit Tests") {                   
                    // runUnitTests(
                    //     [
                    //         projects: findUnitTestProjects(),
                    //         params: "--no-build -c Release --filter Category=Unit"
                    //     ])
                    sh '''
                        echo "Stage: Unit Tests"
                    '''
                }

				stage("Code Analytics") {
                // TBD
                    sh '''
                        echo "Stage: Code Analytics"
                    '''
				}

				stage("Package") {                    
                    // package([solutionPath: "coe_ssa_saas.packagefile", params: ""])                    
					// stash name: "${context.application}-${context.branchName}"
                    sh '''
                        echo "Stage: Package"
                    '''
				}
			}
			finally {
				step([$class: 'WsCleanup', notFailBuild: true])
			}
		}
    }
}
catch (Exception e){
    println "ERROR: '${context.application}-api' branch '${env.BRANCH_NAME}' build #${env.BUILD_NUMBER} failed with error: ${e}. (<${env.BUILD_URL}|Open>)"
    //can we replace this with a Teams send ??  To a group??    
    //slackSend channel: "#${context.application}-api", teamDomain: 'coe_ssa_saassupport', token: '????', color: 'danger', message: "'${context.application}-api' branch '${env.BRANCH_NAME}' build #${env.BUILD_NUMBER} failed with error: ${e}. (<${env.BUILD_URL}|Open>)"    
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

def packageIAM()
{

                // sh '''
                //     set +x # Don't echo credentials from the login command!
                //     eval $(aws ecr get-login --region "$AWS_REGION" --no-include-email)
                // '''
                // withCredentials([string(credentialsId: 'machine-user-github-oauth-token', variable: 'GITHUB_OAUTH_TOKEN')]) {
                //   sh '''
                //       build-docker-image \
                //         --docker-image-name "087285199408.dkr.ecr.us-east-1.amazonaws.com/sample-app-frontend-multi-account-acme" \
                //         --docker-image-tag "$GIT_COMMIT" \
                //         --docker-build-path "." \
                //         --build-arg GITHUB_OAUTH_TOKEN="$GITHUB_OAUTH_TOKEN"
                //   '''
                sh '''
                    echo "packageIAM"
                '''
                }
            
}

def dockerizeMainAPI(def context)
{
    sh '''
        echo "dockerizeMainAPI"
    '''
    // //container registry
    // ecr.createRepository([repositoryName: "${context.application}/api", region: context.region, profile: context.profile ])

    // // Just get docker login string once for all containers
    // dockerci.login([
    //     region: context.region,
    //     profile: context.profile
    // ])

    // dir('src/coe_ssa_saas/publish'){
    //     dockerci.build([
    //         dockerFile: "./Dockerfile", 
    //         name: "${context.application}/api",
    //         path: ".",
    //         params: "--no-cache=true --build-arg source=."
    //     ])
    //     dockerci.tag([
    //         account: context.accountnumber,
    //         region: context.region,
    //         sourceImage: "${context.application}/api",
    //         targetImage: "${context.application}/api",
    //         targetTag: "${context.branchName}-${env.BUILD_NUMBER}"
    //     ])
    //     dockerci.tag([
    //         account: context.accountnumber,
    //         region: context.region,
    //         sourceImage: "${context.application}/api",
    //         targetImage: "${context.application}/api",
    //         targetTag: "latest"
    //     ])
    //     dockerci.push([
    //         account: context.accountnumber,
    //         region: context.region,
    //         name: "${context.application}/api",
    //         image: "${context.application}/api",
    //         tag: "${context.branchName}-${env.BUILD_NUMBER}"
    //     ])
    //     dockerci.push([
    //         account: context.accountnumber,
    //         region: context.region,
    //         name: "${context.application}/api",
    //         image: "${context.application}/api",
    //         tag: "latest"
    //     ])
    // }
}


def runTerraform(InfraModel model, def context, def regionMap) {
    println "In runTerraform - uuid='${context.uuid}'"
    sh '''
        echo "runTerraform"
    '''
// try{
    
//     //what do we need to do here before apply??
//     stage('inittf'){
//         node{
//             withCredentials([[

//             ]]){
//                 ansiColor('xterm'){
//                     sh 'terraform init'
//                 }

//             }
//             }
    
//     }
//     stage('plantf'){
//         node{
//             withCredentials([[
//                 $class: 'AmazonWebServicesCredentialsBinding',
//                 credentialsId: credentialsId,
//                 accessKeyVariable: 'AWS_ACCESS_KEY_ID',
//                 secretKeyVariable:'AWS+SECRET_ACCESS'
//             ]]) {
//                 ansiColor('xterm') {
//                     sh 'terraform plan'
//                 }
//             }
//         }
//     }
//         //only run apply if not a pull request - if approved
//     if (env.BRANCH_NAME == 'master'){
//         //run terraform apply
//         stage('apply'){
//             node{
//                 withCredentials([[
//                 $class: 'AmazonWebServicesCredentialsBinding',
//                 credentialsId: credentialsId,
//                 accessKeyVariable: 'AWS_ACCESS_KEY_ID',
//                 secretKeyVariable:'AWS+SECRET_ACCESS'
//             ]]) {
//                 ansiColor('xterm') {
//                     sh 'terraform apply -auto-approve'
//                 }
//             }
//             }
//         }
//         stage('show'){
//             node{
//                 withCredentials([[
//                 $class: 'AmazonWebServicesCredentialsBinding',
//                 credentialsId: credentialsId,
//                 accessKeyVariable: 'AWS_ACCESS_KEY_ID',
//                 secretKeyVariable:'AWS+SECRET_ACCESS'
//             ]]) {
//                 ansiColor('xterm') {
//                     sh 'go test -v -timeout 30m' //Test the infrastructure is ok
//                     sh 'terraform show'
//                 }
//             }
//             }
//         }
        
//         currentBuild.result = 'SUCCESS'

//     }
//     catch(org.jenkinsci.plugins.workflow.steps.FlowInterruptedException flowError){
//         currentBuild.result='ABORTED'
//     }
//     catch(err){
//         currentBuild.result = 'FAILURE'
//         throw err
//     }
//     finally{
        
//     }

}


//NOTE!!!!! need to replace this with runTerraform - terraform apply
def runTerraformMainAPI(InfraModel model, def context, def regionMap) {
    println "In runTerraformMainAPI - uuid='${context.uuid}'"
    def authEnvironment = getSetting(context.settings, "AuthEnvironment", model.environment)
    def appComponent = "mainapi"

    sh '''
        echo "runTerraformMainAPI"
    '''

    //  //what do we need to do here before apply??
    // stage('inittf'){
    //     node{
    //         withCredentials([[

    //         ]]){
    //             ansiColor('xterm'){
    //                 sh 'terraform init'
    //             }

    //         }
    //         }
    
    // }
    // stage('plantf'){
    //     node{
    //         withCredentials([[
    //             $class: 'AmazonWebServicesCredentialsBinding',
    //             credentialsId: credentialsId,
    //             accessKeyVariable: 'AWS_ACCESS_KEY_ID',
    //             secretKeyVariable:'AWS+SECRET_ACCESS'
    //         ]]) {
    //             ansiColor('xterm') {
    //                 sh 'terraform plan'
    //             }
    //         }
    //     }
    // }
    // //only run apply if not a pull request - if approved
    // if (env.BRANCH_NAME == 'master'){
    //     //run terraform apply
    //     stage('apply'){
    //         node{
    //             withCredentials([[
    //             $class: 'AmazonWebServicesCredentialsBinding',
    //             credentialsId: credentialsId,
    //             accessKeyVariable: 'AWS_ACCESS_KEY_ID',
    //             secretKeyVariable:'AWS+SECRET_ACCESS'
    //         ]]) {
    //             ansiColor('xterm') {
    //                 sh 'terraform apply -auto-approve'
    //             }
    //         }
    //         }
    //     }
    //     stage('show'){
    //         node{
    //             withCredentials([[
    //             $class: 'AmazonWebServicesCredentialsBinding',
    //             credentialsId: credentialsId,
    //             accessKeyVariable: 'AWS_ACCESS_KEY_ID',
    //             secretKeyVariable:'AWS+SECRET_ACCESS'
    //         ]]) {
    //             ansiColor('xterm') {
    //                 sg 'go test -v -timeout 30m'
    //                 sh 'terraform show'
    //             }
    //         }
    //         }
    //     }
        
    //     currentBuild.result = 'SUCCESS'

    // }
    // catch(org.jenkinsci.plugins.workflow.steps.FlowInterruptedException flowError){
    //     currentBuild.result='ABORTED'
    // }
    // catch(err){
    //     currentBuild.result = 'FAILURE'
    //     throw err
    // }
    // finally{
        
    // }

}

def runTerraformStage(InfraModel model, def context, def regionMap) {
    println "In runTerraformStage - uuid='${context.uuid}'"
    def authEnvironment = getSetting(context.settings, "AuthEnvironment", model.environment)

    sh '''
        echo "runTerraformStage"
    '''

//    //what do we need to do here before apply??
//     stage('inittf'){
//         node{
//             withCredentials([[

//             ]]){
//                 ansiColor('xterm'){
//                     sh 'terraform init'
//                 }

//             }
//             }
    
//     }
//     stage('plantf'){
//         node{
//             withCredentials([[
//                 $class: 'AmazonWebServicesCredentialsBinding',
//                 credentialsId: credentialsId,
//                 accessKeyVariable: 'AWS_ACCESS_KEY_ID',
//                 secretKeyVariable:'AWS+SECRET_ACCESS'
//             ]]) {
//                 ansiColor('xterm') {
//                     sh 'terraform plan'
//                 }
//             }
//         }
//     }
//     //only run apply if not a pull request - if approved
//     if (env.BRANCH_NAME == 'master'){
//         //run terraform apply
//         stage('apply'){
//             node{
//                 withCredentials([[
//                 $class: 'AmazonWebServicesCredentialsBinding',
//                 credentialsId: credentialsId,
//                 accessKeyVariable: 'AWS_ACCESS_KEY_ID',
//                 secretKeyVariable:'AWS+SECRET_ACCESS'
//             ]]) {
//                 ansiColor('xterm') {
//                     sh 'terraform apply -auto-approve'
//                 }
//             }
//             }
//         }
//         stage('show'){
//             node{
//                 withCredentials([[
//                 $class: 'AmazonWebServicesCredentialsBinding',
//                 credentialsId: credentialsId,
//                 accessKeyVariable: 'AWS_ACCESS_KEY_ID',
//                 secretKeyVariable:'AWS+SECRET_ACCESS'
//             ]]) {
//                 ansiColor('xterm') {
//                     sh 'go test -v -timeout 30m'
//                     sh 'terraform show'
//                 }
//             }
//             }
//         }
        
//         currentBuild.result = 'SUCCESS'

//     }
//     catch(org.jenkinsci.plugins.workflow.steps.FlowInterruptedException flowError){
//         currentBuild.result='ABORTED'
//     }
//     catch(err){
//         currentBuild.result = 'FAILURE'
//         throw err
//     }
//     finally{
        
//     }


}

def runApps(InfraModel model, def context, def regionMap) {
    def exportName = "ssl-wild-${model.environment}-${context.domainCountryCode}-${context.certSuffix}-SslCertificateId" 
    def certArn = cloudformation.getExportValue([
        exportName: exportName,
        region    : "us-east-1",
        profile   : context.profile
    ])

    sh '''
        echo "runApps"
    '''

//     def domain = cloudformation.getExportValue([
//         exportName: "dns-${model.environment}-${context.domainCountryCode}-${context.certSuffix}-SubZoneDomain",
//         region    : context.region,
//         profile   : context.profile
//     ])

//     println "${exportName}=${certArn}"
//     println "${domain}"

//     //apiResponse = apigateway.updateDomain([
//     //        application: context.application,
//     //        applicationUriRoot: context.applicationUriRoot,
//     //        profile    : context.profile,
//     //        region     : context.region,
//     //        account    : context.account,
//     //        domain     : domain,
//     //        certArn    : certArn,
//     //        environment: model.environment,
//     //])

//    //what do we need to do here before apply??
//     stage('inittf'){
//         node{
//             withCredentials([[

//             ]]){
//                 ansiColor('xterm'){
//                     sh 'terraform init'
//                 }

//             }
//             }
    
//     }
//     stage('plantf'){
//         node{
//             withCredentials([[
//                 $class: 'AmazonWebServicesCredentialsBinding',
//                 credentialsId: credentialsId,
//                 accessKeyVariable: 'AWS_ACCESS_KEY_ID',
//                 secretKeyVariable:'AWS+SECRET_ACCESS'
//             ]]) {
//                 ansiColor('xterm') {
//                     sh 'terraform plan'
//                 }
//             }
//         }
//     }
//     //only run apply if not a pull request - if approved
//     if (env.BRANCH_NAME == 'master'){
//         //run terraform apply
//         stage('apply'){
//             node{
//                 withCredentials([[
//                 $class: 'AmazonWebServicesCredentialsBinding',
//                 credentialsId: credentialsId,
//                 accessKeyVariable: 'AWS_ACCESS_KEY_ID',
//                 secretKeyVariable:'AWS+SECRET_ACCESS'
//             ]]) {
//                 ansiColor('xterm') {
//                     sh 'terraform apply -auto-approve'
//                 }
//             }
//             }
//         }
//         stage('show'){
//             node{
//                 withCredentials([[
//                 $class: 'AmazonWebServicesCredentialsBinding',
//                 credentialsId: credentialsId,
//                 accessKeyVariable: 'AWS_ACCESS_KEY_ID',
//                 secretKeyVariable:'AWS+SECRET_ACCESS'
//             ]]) {
//                 ansiColor('xterm') {
//                     sh 'go test -v -timeout 30m'
//                     sh 'terraform show'
//                 }
//             }
//             }
//         }
        
//         currentBuild.result = 'SUCCESS'

//     }
//     catch(org.jenkinsci.plugins.workflow.steps.FlowInterruptedException flowError){
//         currentBuild.result='ABORTED'
//     }
//     catch(err){
//         currentBuild.result = 'FAILURE'
//         throw err
//     }
//     finally{
        
//     }


}

def loadRegionMap(flags){
    return [
        "us-east-1": [
            name             : "US East (N. Virginia)",
            regionCountryCode: "us",
            bucket           : flags.isReleasableBranch() ? "coe_ssa_saas-artifacts-us-east-1" : "coe_ssa_saas-artifacts-dev-us-east-1",
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
            regionCountryCode: "ca",
            bucket           : "coe_ssa_saas-artifacts-ca-central-1",
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

def findUnitTestProjects()
{
    def filePaths = []
    def files = findFiles(glob: '**/*.Tests')    
    for ( int i = 0; i < files.size(); i++ )     
       filePaths.add(files[i].path)
        
    return filePaths
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
