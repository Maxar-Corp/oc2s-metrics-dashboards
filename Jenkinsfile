String buildNodeDefault = "omar-build"

//noinspection GroovyAssignabilityCheck
properties([
        parameters([
                string(name: 'BUILD_NODE', defaultValue: buildNodeDefault, description: 'The build node to run on'),
                booleanParam(name: 'CLEAN_WORKSPACE', defaultValue: true, description: 'Clean the workspace at the end of the run')
        ]),
        pipelineTriggers([
                [$class: "GitHubPushTrigger"]
        ])
])

// We use the get[] syntax here because the first time a new branch of pipeline is loaded, the property does not exist.
node(params["BUILD_NODE"] ?: buildNodeDefault) {

    stage("Checkout") {
        checkout(scm)
    }


    stage("Generate visualizations") {
        sh "ansible-playbook kibana/generate.yml"
        archiveArtifacts "kibana/output/"
    }

    stage("Clean Workspace") {
        if ("${CLEAN_WORKSPACE}" == "true")
            step([$class: 'WsCleanup'])
    }
}