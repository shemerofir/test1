//def dockerHubUser = "shemerofir"
def usernames = """return[
'shemerofir',
'nirgeier'
]""";

//Script for the branch, you can reference the previous script value witn the "REPO" variable
def credsId = """def credsNames = []

def creds = com.cloudbees.plugins.credentials.CredentialsProvider.lookupCredentials(
      com.cloudbees.plugins.credentials.Credentials.class
)

def credsIds = [];
credsIds.push('None');
for (cred in creds) {
    credsIds.push(cred.id)
}

return credsIds;""";

def repoScript = """import groovy.json.JsonSlurper

def creds = com.cloudbees.plugins.credentials.CredentialsProvider.lookupCredentials(
      com.cloudbees.plugins.credentials.Credentials.class
)

token = 'none'

for (cred in creds) {
    if(cred.id == CREDENTIAL && cred.hasProperty('secret')){
        token = cred.secret
    }
}

def get;
if(token != 'none'){
    get = new URL("https://api.github.com/user/repos").openConnection();
    get.setRequestProperty("Authorization", 'token ' + token);
}else {
    get = new URL("https://api.github.com/users/" + USERNAME +"/repos").openConnection();
}

def getRC = get.getResponseCode();

if (getRC.equals(200)) {
    def json = get.inputStream.withCloseable { inStream ->
        new JsonSlurper().parse( inStream as InputStream )
    }

    def item = json;
    def names = [];

    item.each { repo ->
        names.push(repo.name);
    }   
    return names;
}""";

//Script for the branch, you can reference the previous script value witn the "REPO" variable
def branchScript = """import groovy.json.JsonSlurper

def creds = com.cloudbees.plugins.credentials.CredentialsProvider.lookupCredentials(
      com.cloudbees.plugins.credentials.Credentials.class
)

def token = 'none'

for (cred in creds) {
    if(cred.id == CREDENTIAL && cred.hasProperty('secret')){
        token = cred.secret
    }
}

def getBranches;
getBranches = new URL("https://api.github.com/repos/" + USERNAME +"/" + REPO + "/branches").openConnection();

if(token != 'none'){
    getBranches.setRequestProperty("Authorization", 'token ' + token);
}

def getRCBranches = getBranches.getResponseCode();

if (getRCBranches.equals(200)) {
   def jsonBr = getBranches.inputStream.withCloseable { inStream ->
           new JsonSlurper().parse( inStream as InputStream )
   }

    def itemBr = jsonBr;
    def namesBr = [];

    itemBr.each { branch ->
        namesBr.push(branch.name);
    } 
    return namesBr;
}""";

pipeline {
    agent any
    stages {
        stage('Parameters'){
            steps {
                script {
                properties([
                        //Creating the parameters, make sure you have Active Choice plugin installed
                        parameters([  [$class: 'ChoiceParameter', 
                                //Single combo-box item select type of choice
                                choiceType: 'PT_SINGLE_SELECT', 
                                description: 'Select the github account', 
                                filterLength: 1, 
                                filterable: false, 
                                //Important for identify it in the cascade choice parameter and the params. values
                                name: 'USERNAME', 
                                script: [
                                    $class: 'GroovyScript', 
                                    //Error script
                                    fallbackScript: [
                                        classpath: [], 
                                        sandbox: false, 
                                        script: 
                                            "return['No accounts registered on pipeline']"
                                    ], 
                                    script: [
                                        classpath: [], 
                                        sandbox: false, 
                                        //Calling local variable with the script as a string
                                        script: "${usernames}"
                                        
                                    ]
                                ]
                            ],
                            [$class: 'ChoiceParameter', 
                                //Single combo-box item select type of choice
                                choiceType: 'PT_SINGLE_SELECT', 
                                description: 'Select the credentialID for the Personal Access Token (optional) - Secret Text Type expeceted', 
                                filterLength: 1, 
                                filterable: true, 
                                //Important for identify it in the cascade choice parameter and the params. values
                                name: 'CREDENTIAL', 
                                script: [
                                    $class: 'GroovyScript', 
                                    //Error script
                                    fallbackScript: [
                                        classpath: [], 
                                        sandbox: false, 
                                        script: 
                                            "return['Could not get the credentials IDs']"
                                    ], 
                                    script: [
                                        classpath: [], 
                                        sandbox: false, 
                                        //Calling local variable with the script as a string
                                        script: "${credsId}"
                                        
                                    ]
                                ]
                            ],
                            [$class: 'CascadeChoiceParameter', 
                                //Single combo-box item select type of choice
                                choiceType: 'PT_CHECKBOX', 
                                description: 'Select the Repository from the Dropdown List', 
                                filterLength: 1, 
                                filterable: true, 
                                referencedParameters: 'CREDENTIAL, USERNAME', 
                                //Important for identify it in the cascade choice parameter and the params. values
                                name: 'REPO', 
                                script: [
                                    $class: 'GroovyScript', 
                                    //Error script
                                    fallbackScript: [
                                        classpath: [], 
                                        sandbox: false, 
                                        script: 
                                            "return['Could not get The Repos']"
                                    ], 
                                    script: [
                                        classpath: [], 
                                        sandbox: false, 
                                        //Calling local variable with the script as a string
                                        script: "${repoScript}"
                                        
                                    ]
                                ]
                            ],
                            //Cascade choice, means you can reference other choice values, like in this case, the REPO
                            //Also, re-runs this scripts every time the referenced choice value changes.
                            [$class: 'CascadeChoiceParameter', 
                                choiceType: 'PT_SINGLE_SELECT', 
                                description: 'Select the Branch from the Dropdown List',
                                name: 'BRANCH',                                
                                filterable: true, 
                                //Referencing the repo
                                referencedParameters: 'REPO, CREDENTIAL, USERNAME', 
                                script: 
                                    [$class: 'GroovyScript', 
                                    fallbackScript: [
                                            classpath: [], 
                                            sandbox: false,
                                            script: "return['Could not get Branch from the Repo']"
                                            ],
                                    script: [
                                            classpath: [],
                                            sandbox: false,
                                            //branchScript variable
                                            script: "${branchScript}"
                                    ] 
                                ]
                            ],
                             string(
                                defaultValue: 'ENTER-BRANCH-NAME',
                                name: 'BRANCHTOCREATE',
                                trim: true
                            )
                            
                        ])
                        
                    ])
                    
                }
                
            }
        }
        stage('checkout scm') {
            when{
                expression { params.BRANCHTOCREATE != 'ENTER-BRANCH-NAME'}
            }
            steps {
                echo "${params.BRANCHTOCREATE}"
                echo "${params.USERNAME}"
                echo "${params.REPO}"
                sh "rm -rf ${params.REPO}"
                sh "git clone git@github.com:${params.USERNAME}/${params.REPO}.git"
                sh "git checkout -b ${params.BRANCHTOCREATE}"
                echo "*********branch ${params.BRANCHTOCREATE} created!*************"
                sh "git push git@github.com:${params.USERNAME}/${params.REPO}.git"
            }
        }

    }
}
