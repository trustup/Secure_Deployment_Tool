properties([
parameters([
    [
    $class: 'ChoiceParameter', 
        choiceType: 'PT_SINGLE_SELECT', 
        description: 'Select the docker image to build', 
        filterLength: 1, 
        filterable: false, 
        name: 'Docker_Image', 
        randomName: 'choice-parameter-24910133008274', 
        script:[
            $class: 'GroovyScript', 
            fallbackScript: [
                classpath: [], oldScript: '', 
                sandbox: false, 
                script: 'return [\'ERROR\']'
            ], 
            script:[
                classpath: [], 
                oldScript: '', 
                sandbox: false, 
                script: 
'''import groovy.json.JsonSlurper
URL docker_image_tags_url = new URL ("https://hub.docker.com/v2/repositories/davideias") 
try {
    def URLConnection http_client = docker_image_tags_url.openConnection() as HttpURLConnection
    http_client.setRequestMethod(\'GET\')
    http_client.connect()
    def dockerhub_response = [:]
    if (http_client.responseCode == 200) {
        dockerhub_response = new JsonSlurper().parseText(http_client.inputStream.getText(\'UTF-8\'))
    } else {
        println("HTTP response error")
        System.exit(0)
    }
    def image_tag_list = []
    dockerhub_response.results.each { tag_metadata -> image_tag_list.add(tag_metadata.name)  }  
    return image_tag_list.sort()
} catch(Throwable t){
    return [t.toString()]
}'''
            ]
        ]
    ],


    [
    $class: 'CascadeChoiceParameter', 
        choiceType: 'PT_SINGLE_SELECT', 
        description: 'Select the tag from the Dropdown List', 
        filterLength: 1, 
        filterable: false, 
        name: 'TAG_image', 
        randomName: 'choice-parameter-5631314456178619', 
        referencedParameters: 'Docker_Image', 
        script: [
            $class: 'GroovyScript', 
            fallbackScript: [
                classpath: [], 
                sandbox: false, 
                script: 
                    'return[\'Error\']'
            ], 
            script: [
                classpath: [], 
                sandbox: false, 
                script: 
'''import groovy.json.JsonSlurper
URL docker_image_tags_url = new URL ("https://registry.hub.docker.com/v2/repositories/davideias/${Docker_Image}/tags") 
try {
    def URLConnection http_client = docker_image_tags_url.openConnection() as HttpURLConnection
    http_client.setRequestMethod('GET')
    http_client.connect()
    def dockerhub_response = [:]
    if (http_client.responseCode == 200) {
        dockerhub_response = new JsonSlurper().parseText(http_client.inputStream.getText('UTF-8'))
    } else {
        println("HTTP response error")
        System.exit(0)
    }
    def image_tag_list = []
    dockerhub_response.results.each { tag_metadata -> image_tag_list.add(tag_metadata.name)  }  
    return image_tag_list.sort()
} catch(Throwable t){
    return [t.toString()]
}
'''
            ]
        ]
    ],

    [
    $class: 'StringParameterDefinition',
    description: 'Add the options for \'docker run\' command (ex \'--name=example -p 8080:8080\')', 
    name: 'Docker_run_options',
    defaultValue: '',
    ],

[
    $class: 'ChoiceParameter', 
    choiceType: 'PT_SINGLE_SELECT', 
    description: 'Select the container type:', 
    filterLength: 1, 
    filterable: false, 
    name: 'Container_Deploymemt', 
    randomName: 'choice-parameter-2632125668535', 
    script: [
        $class: 'GroovyScript', 
        fallbackScript: [classpath: [], oldScript: '', sandbox: false, script: 
        ''], 
        script: [classpath: [], oldScript: '', sandbox: true, script: 
        'return [\'SGX-Secured_container\', \'native_container\']'
        ]
    ]
],

])])



node {
    stage('Example') {
        def image = "davideias/${Docker_Image}:${TAG_image}"
        def SSH="ssh trustup@10.0.0.4"
        if ("${Container_Deploymemt}" == 'native_container'){

            sh "docker pull ${image}"
            sh "docker run  ${Docker_run_options} ${image}"

        }else{
            
            def fileBase64 = input message: 'Enter the ManifestFile for image configuration', parameters: [base64File('ManifestFile')]
            withEnv(["fileBase64=$fileBase64"]) {
                sh "echo $fileBase64 | base64 -d > ManifestFile.manifest"
                //sh 'base64 --decode > ManifestFile.manifest'
                
                // powershell '[IO.File]::WriteAllBytes("myFile.txt", [Convert]::FromBase64String($env:fileBase64))'
            }
            
            sh "scp ManifestFile.manifest trustup@sgx.trustup.it:/home/trustup/gsc"
            sh "$SSH docker pull ${image}"   
            sh "$SSH 'cd gsc && ./gsc build --insecure-args ${image} ManifestFile.manifest --rm\
            && openssl genrsa -3 -out key.pem 3072 \
            && ./gsc sign-image ${image} key.pem \
            && docker run --device=/dev/sgx_enclave --rm  ${Docker_run_options} -v /var/run/aesmd/aesm.socket:/var/run/aesmd/aesm.socket gsc-${image} '" 
           // && docker rmi -f  ${image} gsc-${image}-unsigned gsc-${image}  ' "

        }
    }
}
