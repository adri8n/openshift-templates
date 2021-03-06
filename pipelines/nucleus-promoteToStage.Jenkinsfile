def templateName = "nucleus"
def templatePath = "https://raw.githubusercontent.com/adri8n/openshift-templates/master/templates/nucleus.json"
def appName = "nucleus-web"
def devProjectName = "dev"
def stageProjectName = "stage"

pipeline {
    agent none
     options {
        // set a timeout of 20 minutes for this pipeline
        timeout(time: 20, unit: 'MINUTES')
    }
    stages {
        stage('Rollout to STAGE') {
            steps {
                script {
                    def nucleus
                    def objs
                    def timestamp
                    openshift.verbose()
                    openshift.withCluster() {
                        openshift.withProject(devProjectName) {
                            nucleus = openshift.selector('dc', [ app : 'nucleus' ])
                            objs = nucleus.objects (exportable:true)
                            secrets = openshift.selector('secrets', [ app : 'nucleus' ]).objects(exportable:true)
                            services = openshift.selector('services', [ app : 'nucleus' ]).objects(exportable:true)
                            timestamp = "${System.currentTimeMillis()}"
                            for ( obj in objs ) {
                                obj.metadata.labels[ "promoted-on" ] = timestamp
                            }
                        }
                        openshift.withProject(stageProjectName) {
                            nucleus.delete('--ignore-not-found')
                            try {
                                openshift.selector('secrets', 'nucleus-web').delete()
                            } catch(Exception ex) {
                                echo "error, probably didn't exist ${ex}"
                            }
                            openshift.create(secrets)
                            try {
                                openshift.create(services)
                            } catch(Exception ex) {
                                echo "error, probably already exists ${ex}"
                            }
                            try {
                                openshift.create('https://raw.githubusercontent.com/adri8n/openshift-templates/master/templates/nucleus-pvcs.json')
                            } catch(Exception ex) {
                                echo "error, probably already exists ${ex}"
                            }
                            openshift.create(objs)
                            try {
                                openshift.selector("service", "nucleus-web-nginx").expose()
                            } catch(Exception ex) {
                                echo "Expose already exists: ${ex}"
                            }
                            openshift.selector("dc", appName).related('pods').untilEach(1) {
                                return (it.object().status.phase == "Running")
                            }
                        }
                    }
                } 
            }
        }
    }
}