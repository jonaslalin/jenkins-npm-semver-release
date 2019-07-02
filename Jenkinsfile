properties([
    disableConcurrentBuilds()
])

def label = "nodejs-${UUID.randomUUID().toString()}"
podTemplate(
    label: label,
    cloud: "openshift",
    serviceAccount: "jenkins",
    containers: [
        containerTemplate(
            name: "jnlp",
            image: "registry.redhat.io/openshift3/jenkins-agent-nodejs-8-rhel7:v3.11",
            alwaysPullImage: true,
            args: "\${computer.jnlpmac} \${computer.name}",
            resourceLimitCpu: "500m",
            resourceLimitMemory: "250Mi"
        )
    ]
) {
    node(label) {
        stage("Checkout") {
            checkout(scm)
        }

        stage("Install") {
            sh("npm ci")
        }

        stage("Test") {
            sh("npm test")
        }

        if (env.BRANCH_NAME == "master") {
            stage("Prepare version") {
                def nextPatchVersion = sh(
                    script: "npm --no-git-tag-version version patch",
                    returnStdout: true
                ).trim()
                def timestamp = sh(
                    script: "TZ=Europe/Stockholm date +%Y%m%d%H%M%S",
                    returnStdout: true
                ).trim()
                def newVersion = "${nextPatchVersion}-alpha.${timestamp}"
                sh("npm --no-git-tag-version version ${newVersion}")
            }
        }

        stage("Build") {
            sh("npm run build")
        }

        if (env.BRANCH_NAME == "master" || env.TAG_NAME) {
            stage("Publish") {
                withCredentials([
                    string(credentialsId: "npm-secret", variable: "NPM_SECRET")
                ]) {
                    withEnv([
                        "npm_config__auth=${env.NPM_SECRET}",
                        "npm_config_email=youremail@email.com",
                        "npm_config_always_auth=true"
                    ]) {
                        sh("npm publish")
                    }
                }
            }
        }
    }
}
