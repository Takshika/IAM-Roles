node {
    // Checkout source code from SCM (Git, etc.)
    checkout(scm)

    // Load environment variables from a properties file based on the current branch name
    loadEnvironmentVariables("parameters/${BRANCH_NAME}.properties")

    // Use Jenkins credentials plugin to inject Vault username and password securely
    withCredentials([usernamePassword(credentialsId: 'vault', passwordVariable: 'VAULT_PASSWORD', usernameVariable: 'VAULT_USER')]) {

        stage('Build environment') {
            // Run ansible playbook with "prepare" tag for environment setup, passing env variable
            sh 'ansible-playbook site.yml -e env=$environment --tags "prepare"'
        }

        stage('Validate CF Templates') {
            // Run ansible playbook with "validate" tag to validate CloudFormation templates
            sh 'ansible-playbook site.yml -e env=$environment --tags "validate"'
        }

        stage('Deploy CF Templates') {
            // Run ansible playbook with "deploy" tag to deploy CloudFormation templates
            sh 'ansible-playbook site.yml -e env=$environment --tags "deploy"'
        }

        stage('Delete CF Templates') {
            // Conditionally run ansible playbook with "delete" tag if DELETE_STACK environment variable is YES
            if (env.DELETE_STACK == 'YES') {
                sh 'ansible-playbook site.yml -e env=$environment --tags "delete"'
            }
        }
    }
}

// Function to load key=value pairs from a properties file into Jenkins environment variables
def loadEnvironmentVariables(path) {
    def props = readProperties file: path   // Read properties file
    keys = props.keySet()                    // Get all keys
    for (key in keys) {
        value = props["${key}"]
        env."${key}" = "${value}"           // Set each key-value as environment variable
    }
}
