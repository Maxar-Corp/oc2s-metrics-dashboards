String buildNodeDefault = "omar-build"

//noinspection GroovyAssignabilityCheck
properties([
        parameters([
                string(name: 'BUILD_NODE', defaultValue: buildNodeDefault, description: 'The build node to run on'),
                string(name: 'indexPattern', defaultValue: buildNodeDefault, description: 'The Kibana index pattern to build into the visualizations'),
                booleanParam(name: 'validateJsonOutput', defaultValue: true, description: 'Checks the output is valid JSON. Disable this to save a few seconds'),
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

    String extraVars = "index_pattern=$indexPattern"
    extraVars += validateJsonOutput ? " validate=true" : '"'

    stage("Generate visualizations") {
        sh "ansible-playbook kibana/generate.yml --extra-vars \"$extraVars\""
        archiveArtifacts "kibana/output/"
    }

    stage("Clean Workspace") {
        if (params["CLEAN_WORKSPACE"] == "true")
            step([$class: 'WsCleanup'])
    }
}