/*
 * creates the job-dsl-templates for all repositories found in the given gogs instance
 */
def createRepoTemplates(gogsUrl, gogsUser) {
    // query repositories
    try {
        response = httpRequest url: "${gogsUrl}/api/v1/user/repos", quiet: true, acceptType: 'APPLICATION_JSON', customHeaders: [[maskValue: true, name: 'Authorization', value: GOGS_USR_TOKEN]]
        repos = readJSON text: response.content
    } catch (err) {
        error("Could not query repositories on the gogs server '${gogsUrl}'")
    }

    // read template content and create job-dsl-template script for each
    def templateContent = readFile file: 'template.txt', encoding: 'utf-8'
    repos.eachWithIndex { repo, index ->
        if (!repo.name.contains('seed')) {
            // create template
            def cloneUrl = repo.clone_url.replaceAll(/^http[s]{0,1}:\/\/([^\/])+/, gogsUrl)
            def currentItemContent = templateContent.replace('__NAME__', repo.name)
                                                    .replace('__DESCRIPTION__', repo.description)
                                                    .replace('__URL__', cloneUrl)
            writeFile encoding: 'utf-8', file: "job_${index}.groovy", text: currentItemContent
            echo "Created seed job template for '${repo.name}'"

            // set webhook if needed
            response = httpRequest url: "${gogsUrl}/api/v1/repos/${gogsUser}/${repo.name}/hooks", quiet: true, acceptType: 'APPLICATION_JSON', customHeaders: [[maskValue: true, name: 'Authorization', value: GOGS_USR_TOKEN]]
            def existingWebhooks = readJSON text: response.content
            // assume that any existing webhook is fine (too lazy to check the URL)
            if (existingWebhooks.size() == 0) {
                def payload = "{ \"type\": \"gogs\",\"active\":true, \"config\": { \"url\": \"${JENKINS_URL}/gogs-webhook/?job=${repo.name}\", \"content_type\": \"json\" } }"
                httpRequest httpMode: 'POST', requestBody: payload, url: "${gogsUrl}/api/v1/repos/${gogsUser}/${repo.name}/hooks", quiet: true, acceptType: 'APPLICATION_JSON', customHeaders: [[maskValue: true, name: 'Authorization', value: GOGS_USR_TOKEN]], contentType: 'APPLICATION_JSON'
                echo "Created a webhook for '${repo.name}'"
            } else {
                echo "No need to create a webhook for '${repo.name}', it already has one."
            }
        }
    }
}

/*
 * Main Pipeline
 *
 * This example assumes:
 * - a secret-text credential called 'gogs-token-thabok' with the value 'token ********************************'
 * - the gogs server to be found at 192.168.19.112:3000
 * - you're looking for repositories of the user 'thabok' inside gogs
 * - the gogs-webhook plugin is installed for Jenkins for the webhooks to work
 */
pipeline {
    agent any 
    stages {
        stage('Pipeline to seed or update all pipelines') {
            steps {
                // create job-dsl-templates for each repo
                withCredentials([string(credentialsId: 'gogs-token-thabok', variable: 'GOGS_USR_TOKEN')]) {
                    script { createRepoTemplates('http://192.168.19.112:3000', 'thabok') }
                }
                
                // create jobs for each repo based on templates
                jobDsl  targets: ['*.groovy'].join('\n')
            }
        }
    }
}
