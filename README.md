
#This is a sample template to build and compile  Java + Maven based components.
#In order to change and add additional configuration  [follow this doc] ( https://cicd-docs.devops.csco.cloud/)

pipeline:
  build_unit_test:
    image: containers.cisco.com/servicescf-devops/maven-cisco-docker
    pull: true
    scripts:
      - mvn -B clean package findbugs:findbugs 
    when:
      event: [push, pull_request]

#Before adding this step make sure the key creation has been done 
  sonar_analysis:
    image: containers.cisco.com/servicescf-devops/drone-sonar
    pull: true
    host: https://engci-sonar-sjc.cisco.com/sonar # Sonar host
    jacoco_reportPath: target/jacoco.exec #[Mandatory when using jacoco] For multimodule project add comma seperated <modulename1>/target/jacoco.exec,<modulename2>/target/jacoco.exec  
    secrets: [ sonar_login ] # Sonar login token
    when:
      event: [pull_request, push] # The events sonar needs to be triggered

  git-secret-scan:
    image: containers.cisco.com/servicescf-devops/drone-git-secret-scan
    pull: true
    environment:
      - SCM_URL=https://bitbucket-eng-sjc1.cisco.com/bitbucket # Same for all Bitbucket repositories
      - RAISE_ERROR=True # [Optional, default false] : will fail the build if any secrets found, note this will be by default true in future
      # - EXCLUDE_FILE_PATTERN="*.asc,*.git-id" #[Optional] Files extension patterns to be excluded
    secrets: [ bitbucket_login ] # Bitbucket token, by default enabled for all repositories, if not present contact us
    when:
      event: [pull_request] # Need to enable with Pull Request
    
