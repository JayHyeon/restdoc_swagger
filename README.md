import org.hidetake.gradle.swagger.generator.GenerateSwaggerUI

import org.springframework.boot.gradle.tasks.bundling.BootJar


buildscript {

    ext {
    
        commonsLang3Version = '3.12.0'
        
        commonsCollections4Version = '4.4'
        
        restdocsApiSpecVersion = '0.16.2'
        
    }
    
}


plugins {

    id 'java'
    
    id 'org.springframework.boot' version '2.7.13-SNAPSHOT'
    
    id 'io.spring.dependency-management' version '1.0.15.RELEASE'
    
    id 'com.epages.restdocs-api-spec' version '0.16.2'
    
    id 'org.hidetake.swagger.generator' version '2.19.2'
    
    id 'jacoco'
    
}


group = 'com.hjh'

sourceCompatibility = '17'


configurations {

    compileOnly {
    
        extendsFrom annotationProcessor
        
    }
    
}


repositories {

    mavenCentral()
    
    maven { url 'https://repo.spring.io/milestone' }
    
    maven { url 'https://repo.spring.io/snapshot' }
    
}


dependencies {

    implementation 'org.springframework.boot:spring-boot-starter-data-redis'
    
    implementation 'org.springframework.boot:spring-boot-starter-validation'
    
    implementation 'org.springframework.boot:spring-boot-starter-web'
    
    implementation 'org.springframework.boot:spring-boot-starter-actuator'
    
    compileOnly 'org.projectlombok:lombok'
    
    developmentOnly 'org.springframework.boot:spring-boot-devtools'
    
    annotationProcessor 'org.springframework.boot:spring-boot-configuration-processor'
    
    annotationProcessor 'org.projectlombok:lombok'
    

    // apache commons lang
    
    implementation "org.apache.commons:commons-lang3:${commonsLang3Version}"
    
    implementation "org.apache.commons:commons-collections4:${commonsCollections4Version}"
    

    // swagger-ui, rest doc
    
    developmentOnly 'org.springframework.boot:spring-boot-devtools'
    
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
    
    swaggerUI 'org.webjars:swagger-ui:4.15.5'
    
    testImplementation "org.springframework.restdocs:spring-restdocs-mockmvc"
    
    testImplementation "com.epages:restdocs-api-spec-mockmvc:${restdocsApiSpecVersion}"
    
    testImplementation 'org.springframework.security:spring-security-test'
    
    implementation "org.springframework.security:spring-security-oauth2-client"
    
}


tasks.named('test') {

    useJUnitPlatform()
    
}


test {

    useJUnitPlatform()
    
//    finalizedBy 'jacocoTestReport'

}


test.doFirst {

    println('---------------- removing old snippets ----------------')
    
    delete file('build/generated-snippets')
    

//    println('---------------- removing old docs     ----------------') // build가 성공해야 swagger 파일 작성해주는 코드. 넣어둔 템플릿을 적용하기 위해 비활성화.

//    delete file('src/main/resources/static/docs')

}


jacoco {

    toolVersion = '0.8.8'
    
}


jacocoTestReport {

    dependsOn ':test'
    

    reports {
    
        xml.enabled true
        
        html.enabled true
        
        csv.enabled false
        
    }


    afterEvaluate {
    
        classDirectories.setFrom(files(classDirectories.files.collect {
        
            fileTree(dir: it,
            
                exclude: [
                
                    'com/hjh/api/ng2/Ng2Application*',
                    
                    'com/hjh/api/core/**'
                    
                ]
                
            )
            
        }))
        
    }
    
}


jacocoTestCoverageVerification {

    violationRules {
    
        rule {
        
            enabled = true
            
            element = 'CLASS'
            

            limit {
            
                counter = 'BRANCH'
                
                value = 'COVEREDRATIO'
                
                minimum = 0.00
                
            }

            limit {
            
                counter = 'LINE'
                
                value = 'COVEREDRATIO'
                
                minimum = 0.00
                
            }


            excludes = [
            
                'com.hjh.api.ng2.Ng2Application',
                
                'com.hjh.api.ng2.model.dto.*',
                
                'com.hjh.api.ng2.model.vo.*',
                
                'com.hjh.api.core.*'
                
            ]
            
        }
        
    }
    
}


openapi3 {

    title = 'Ng2'
    
    servers = [{ url = "https://api.ng2.com" }]
    
    description = 'Ng2 API Server.'
    
    version = '0.0.1'
    
    format = 'yaml'
    
}


swaggerSources {

    api {
    
        setInputFile(file("${project.buildDir}/api-spec/openapi3.yaml"))
        
    }
    
}


tasks.withType(GenerateSwaggerUI) {

    dependsOn 'openapi3'
    
}


tasks.register('copySwaggerUI', Copy) {

//    dependsOn 'generateSwaggerUIApi'

//
//    def generateSwaggerUIApiTask = tasks.named('generateSwaggerUIApi', GenerateSwaggerUI).get() // generateSwaggerUIApiTask로 생성된 문서를 build 결과에 넣을 수 있지만 넣어둔 템플릿을 적용하기 위해 비활성화.

//
//    from("${generateSwaggerUIApiTask.outputDir}")

    from("src/main/resources/static/docs")
    
    into("${project.buildDir}/resources/main/static/swagger") // docs -> swagger 폴더명 변경으로 주소 변경. 기존 /docs/index.html -> 변경 /swagger/index.html
    
//
//    from("${generateSwaggerUIApiTask.outputDir}")

//    into('src/main/resources/static/docs')

}


tasks.withType(BootJar) {

    dependsOn 'copySwaggerUI'
    
}

bootJar {

    dependsOn(':openapi3', ':jacocoTestReport', ':jacocoTestCoverageVerification')
    

    tasks['jacocoTestReport'].mustRunAfter(tasks['test'])
    
    tasks['jacocoTestCoverageVerification'].mustRunAfter(tasks['jacocoTestReport'])
    

    from('src/main/resources/static') {
    
        into 'static'
        
    }
    
}
