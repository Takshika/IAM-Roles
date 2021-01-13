node {

    checkout(scm)
    loadEnvironmentVariables("parameters/${BRANCH_NAME}.properties")
    withCredentials([usernamePassword(credentialsId: 'vault', passwordVariable: 'VAULT_PASSWORD', usernameVariable: 'VAULT_USER')]) {

    stage ('Build environment'){
    sh 'ansible-playbook site.yml -e env=$environment  --tags "prepare"'
    }

    stage ('Validate CF Templates'){
    sh 'ansible-playbook site.yml -e env=$environment  --tags "validate"'
    }

    stage ('Deploy CF Templates'){
    sh 'ansible-playbook site.yml -e env=$environment  --tags "deploy"'  
    }

    stage ('Delete CF Templates'){
    if (env.DELETE_STACK == 'YES'){
    sh 'ansible-playbook site.yml -e env=$environment  --tags "delete"'
    }  
    }
}
}

def loadEnvironmentVariables(path){
    def props = readProperties  file: path
    keys= props.keySet()
    for(key in keys) {
        value = props["${key}"]
        env."${key}" = "${value}"
    }
}