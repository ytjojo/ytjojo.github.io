
---
title: lint工作流程研究
date: 2018-03-16 16:48:20
categories: lint
tags: [lint,源码分析，androidLint]
---

>文章会试图解答如下问题
>1. lintOptions如何获取
>2. 系统定义Issues如何被加载进来的
>3. 用户自定lint.jar如何被找到的
>4. Detector如何加载的


### lint总体流程

1. 创建AndroidProject
2. 获得LintOptions配置信息
3. 获得系统定义的IssueRegistry，为BuiltinIssueRegistry的子类
3. 获得所有lint.jar
4. 获得所有定义的Issues，包括系统和用户定义的
5. 获得检测范围Scope
6. 获得Detectors，获取lint.xml配置信息
7. checkProject检查当前工程和所有依赖工程
8. runFileDetectors

### lint入口


gradle 任务的入口
入口类为com.android.build.gradle.tasks.Lint.groovy



### 获得project
lint()方法中,先创建AndroidProject对象
首先判断是单个变体还是所有变体
如果命令gradle app:lint则是所有变体
如果命令是radle app:lintDebug则值检查debug变体

```
@TaskAction
    public void lint() throws IOException {
        AndroidProject modelProject = createAndroidProject(getProject());
        if (getVariantName() != null && !getVariantName().isEmpty()) {
            for (Variant variant : modelProject.getVariants()) {
                if (variant.getName().equals(getVariantName())) {
                    lintSingleVariant(modelProject, variant);
                }
            }
        } else {
            lintAllVariants(modelProject);
        }
    }
```
如何获得变体呢，这就是createAndroidProject的作用了。进入该方法，方法内部又调用ToolingModelBuilder的buildAll（）方法
buildAll调用buildAndroidProject（），在这里会根据buildType和productFlavors生成各种变体，获得各种配置信息，特别是LintOptions
为后来lint检查做准备。这信息最终保存在DefaultAndroidProject中，DefaultAndroidProject最终赋值给lint()方法的临时变量modelProject中。

```
 private Object buildAndroidProject(Project project) {
        Integer modelLevelInt = AndroidGradleOptions.buildModelOnlyVersion(project);
        if (modelLevelInt != null) {
            modelLevel = modelLevelInt;
        }
        modelWithFullDependency = AndroidGradleOptions.buildModelWithFullDependencies(project);

        // Get the boot classpath. This will ensure the target is configured.
        List<String> bootClasspath = androidBuilder.getBootClasspathAsStrings(false);

        List<File> frameworkSource = Collections.emptyList();

        // List of extra artifacts, with all test variants added.
        List<ArtifactMetaData> artifactMetaDataList = Lists.newArrayList(
                extraModelInfo.getExtraArtifacts());

        for (VariantType variantType : VariantType.getTestingTypes()) {
            artifactMetaDataList.add(new ArtifactMetaDataImpl(
                    variantType.getArtifactName(),
                    true /*isTest*/,
                    variantType.getArtifactType()));
        }
		//根据gradle文件中配置的lintOptions获得配置结果，保存在LintOptions这个类中
        LintOptions lintOptions = com.android.build.gradle.internal.dsl.LintOptions.create(
                config.getLintOptions());

        AaptOptions aaptOptions = AaptOptionsImpl.create(config.getAaptOptions());

        List<SyncIssue> syncIssues = Lists.newArrayList(extraModelInfo.getSyncIssues().values());

        List<String> flavorDimensionList = config.getFlavorDimensionList() != null ?
                config.getFlavorDimensionList() : Lists.newArrayList();

        toolchains = createNativeToolchainModelMap(ndkHandler);

        ProductFlavorContainer defaultConfig = ProductFlavorContainerImpl
                .createProductFlavorContainer(
                        variantManager.getDefaultConfig(),
                        extraModelInfo.getExtraFlavorSourceProviders(
                                variantManager.getDefaultConfig().getProductFlavor().getName()));

        Collection<BuildTypeContainer> buildTypes = Lists.newArrayList();
        Collection<ProductFlavorContainer> productFlavors = Lists.newArrayList();
        Collection<Variant> variants = Lists.newArrayList();

		//获取配置的buildType
        for (BuildTypeData btData : variantManager.getBuildTypes().values()) {
            buildTypes.add(BuildTypeContainerImpl.create(
                    btData,
                    extraModelInfo.getExtraBuildTypeSourceProviders(btData.getBuildType().getName())));
        }
  //获取配置的ProductFlavors
        for (ProductFlavorData pfData : variantManager.getProductFlavors().values()) {
            productFlavors.add(ProductFlavorContainerImpl.createProductFlavorContainer(
                    pfData,
                    extraModelInfo.getExtraFlavorSourceProviders(pfData.getProductFlavor().getName())));
        }
//生成所有组合后的变体，如ProductFlavors 为feature1 则变体为feature1Debug 和feature1Release
        for (BaseVariantData<? extends BaseVariantOutputData> variantData : variantManager.getVariantDataList()) {
            if (!variantData.getType().isForTesting()) {
                variants.add(createVariant(variantData));
            }
        }

        return new DefaultAndroidProject(
                Version.ANDROID_GRADLE_PLUGIN_VERSION,
                project.getName(),
                defaultConfig,
                flavorDimensionList,
                buildTypes,
                productFlavors,
                variants,
                androidBuilder.getTarget() != null ? androidBuilder.getTarget().hashString() : "",
                bootClasspath,
                frameworkSource,
                cloneSigningConfigs(config.getSigningConfigs()),
                aaptOptions,
                artifactMetaDataList,
                findUnresolvedDependencies(syncIssues),
                syncIssues,
                config.getCompileOptions(),
                lintOptions,
                project.getBuildDir(),
                config.getResourcePrefix(),
                ImmutableList.copyOf(toolchains.values()),
                config.getBuildToolsVersion(),
                projectType,
                Version.BUILDER_MODEL_API_VERSION,
                generation);
    }
```
![lintOptions](./pic/lintoptions.png)

获得androidProject后进入runLint()这个方法，进入下一个环节
### 获取系统定义IssueRegistry，

在此方法中主要功能是做配置和初始化核心类

在runlint方法中首先通过createIssueRegistry获得系统的IssueRegistry
该类为LintGradleIssueRegistry 继承自BuiltinIssueRegistry
BuiltinIssueRegistry的静态代码块添加所有系统定义好的Issues，总共293个
并在getIssues()方法中返回Issues集合，这就是系统定义Issue来源
再次回到runLint方法中
### 初始化LintGradleClient，并执行client.run()
该方法主要是四个功能
初始化IssueRegistry，
syncOptions同步LintOptions
初始化LintGradleClient
执行client.run(registry)
最关键的是client.run(registry)，最终调用lintDrver的analysis方法
```java
private Pair<List<Warning>,LintBaseline> runLint(
 		 IssueRegistry registry = createIssueRegistry();
         LintCliFlags flags = new LintCliFlags();
         LintGradleClient client = new LintGradleClient(registry, flags, getProject(), modelProject,
                sdkHome, variant, getBuildTools());

  //...
 		try {
            warnings = client.run(registry);
        } catch (IOException e) {
            throw new GradleException("Invalid arguments.", e);
        }
}
```
client.run(registry)方法最终走到`public int run(@NonNull IssueRegistry registry, @NonNull List<File> files)`这个方法

### 初始化LintDriver
```
    public int run(@NonNull IssueRegistry registry, @NonNull List<File> files) throws IOException {
        assert !flags.getReporters().isEmpty();
        this.registry = registry;
        driver = new LintDriver(registry, this);

        driver.setAbbreviating(!flags.isShowEverything());

       
		//调用driver启动分析
        driver.analyze(createLintRequest(files));

        ...

        return flags.isSetExitCode() ? (hasErrors ? ERRNO_ERRORS : ERRNO_SUCCESS) : ERRNO_SUCCESS;
    }
```
在这个方法中可以看到初始化LintDriver,并调用driver.analyze(createLintRequest(files))
其中还有个关键方法createLintRequest(files)
### 创建LintRequest类
```
 protected LintRequest createLintRequest(@NonNull List<File> files) {
        LintRequest lintRequest = new LintRequest(this, files);
        if (Lint.MODEL_LIBRARIES) {
            LintGradleProject.ProjectSearch search = new LintGradleProject.ProjectSearch();
            Project project = search.getProject(this, gradleProject, variant.getName());
            lintRequest.setProjects(Collections.singletonList(project));
            setCustomRules(search.customViewRuleJars);
        } else {
            Pair<LintGradleProject,List<File>> result = LintGradleProject.create(
                    this, modelProject, variant, gradleProject);
            lintRequest.setProjects(Collections.singletonList(result.getFirst()));
            setCustomRules(result.getSecond());
        }

        return lintRequest;
    }
```

此方法有三个作用

1. org.gradle.api.Project转换为com.android.tools.lint.detector.api.Project,并赋值给lintRequest
2. 搜集放在工程或者第三方依赖的lint.jar
2. 将搜集的lint.jar集合复制给lintClient的customRules
3. 生成LintRequest对象

如何搜集lint.jar
1. 先检查当前Project的build目录
如果/build/lint/lint.jar存在则将File保存在customViewRuleJars中
2. 从依赖的项目获取lint.jar可以是工程，也可以是第三方库中aar中lint.jar
第三方库lint.jar一般会放入build-cache中
如C:\Users\Administrator\.android\build-cache\0c1baf206b683783b1aeb51ee8d63025f569746d\output\jars\lint.jar
依赖的工程的lint.jar地址E:\github\LintRulesForAndroid\module_news\build\intermediates\bundles\default\lint.jar

在createLintRequest方法中 创建LintGradleProject.ProjectSearch对象，调用其getProject方法
在此方法把找到lint.jar放入customViewRuleJars这个数组中然后调用LintGradleClient的setCustomRules把lint.jar数组赋值给 给customRules，进入getProject()查看获得lint.jar的源码
```
public Project getProject(
                @NonNull LintGradleClient client,
                @NonNull AndroidProject project,
                @NonNull Variant variant,
                @NonNull org.gradle.api.Project gradleProject) {
            Project cached = appProjects.get(project);
            if (cached != null) {
                return cached;
            }
            mSeen.add(project);
            File dir = gradleProject.getProjectDir();
            AppGradleProject lintProject = new AppGradleProject(client, dir, dir, project, variant);
            appProjects.put(project, lintProject);
			//build/lint/lint.jar判断是否存在，存在就保存到customViewRuleJars
            File appLintJar = new File(gradleProject.getBuildDir(),
                    "lint" + separatorChar + "lint.jar");
            if (appLintJar.exists()) {
                customViewRuleJars.add(appLintJar);
            }

            // DELIBERATELY calling getDependencies here (and Dependencies#getProjects() below) :
            // the new hierarchical model is not working yet.
            Dependencies dependencies = variant.getMainArtifact().getDependencies();
            for (AndroidLibrary library : dependencies.getLibraries()) {
                if (library.getProject() != null) {
                    // Handled below

                    // ...except in that case we don't find custom rule jars (since those are
                    // tied to the AndroidLibrary); include those here.
                    File ruleJar = library.getLintJar();
					// 第三方库lint.jar是否存在
					//C:\Users\Administrator\.android\build-cache\0c1baf206b683783b1aeb51ee8d63025f569746d\output\jars\lint.jar
                    //E:\github\LintRulesForAndroid\module_news\build\intermediates\bundles\default\lint.jar
                    if (ruleJar.exists()) {
                        customViewRuleJars.add(ruleJar);
                    }

                    continue;
                }
                lintProject.addDirectLibrary(getLibrary(client, library, gradleProject, variant));
            }

          


            return lintProject;
        }


```
### lintDriver.analyze()
创建完LintRequet，此后开始进入LintDriver的analyze()方法，这才是真正lint启动的地方
```
 private void analyze() {

        scope = request.getScope();
        assert scope == null || !scope.contains(Scope.ALL_RESOURCE_FILES) ||
                scope.contains(Scope.RESOURCE_FILE);
        Collection<Project> projects;
        try {
			//获得需要检测的Project
            projects = request.getProjects();
            if (projects == null) {
                projects = computeProjects(request.getFiles());
            }
        } catch (CircularDependencyException e) {
            
        }
        if (projects.isEmpty()) {
            client.log(null, "No projects found for %1$s", request.getFiles().toString());
            return;
        }
        if (canceled) {
            return;
        }
		//找到所有的lint.jar并获得系统和用户自定义ISsues
        registerCustomDetectors(projects);

        if (scope == null) {
            scope = Scope.infer(projects);
        }


        for (Project project : projects) {
            phase = 1;

            Project main = request.getMainProject(project);

            // The set of available detectors varies between projects
			//根据ISsues获得所有Detectors
            computeDetectors(project);

            if (applicableDetectors.isEmpty()) {
                // No detectors enabled in this project: skip it
                continue;
            }

            checkProject(project, main);
            if (canceled) {
                break;
            }

            runExtraPhases(project, main);
        }

      
    }
```


然后从LintRequest对象获取分析的project，如果为空通过computeProjects这个方法来找到需要扫描的Projet
什么时候为空呢？ 如果直接输入命令 gradle lint 此时获得的projects为空，如果命令是gradle app:lint 则Projects为数组长度为1，仅包含app这个Project


原理就是根据扫描File列表调用 client.getProject(projectDir, rootDir)来获得Project对象
如果发现循环依赖会报CircularDependencyException，例如 A 依赖B B依赖C,C又依赖A
如果发现projects列表为空或者被用户取消则终止分析
### 获得系统和用户定义Issues

下一步是registerCustomDetectors，通过lint.jar获得用户自定义Issue。这也是为什么lint.jar要放在build\intermediates\lint目录下
或者用户目录下.android/lint目录会生效

这段代码是

```java
 
 private void registerCustomDetectors(Collection<Project> projects) {
        // Look at the various projects, and if any of them provide a custom
        // lint jar, "add" them (this will replace the issue registry with
        // a CompositeIssueRegistry containing the original issue registry
        // plus JarFileIssueRegistry instances for each lint jar
        Set<File> jarFiles = Sets.newHashSet();
        for (Project project : projects) {
            jarFiles.addAll(client.findRuleJars(project));
            for (Project library : project.getAllLibraries()) {
                jarFiles.addAll(client.findRuleJars(library));
            }
        }
		//从用户目录.android/lint目录获取lint.jar
		//我电脑上路径是C:/Users/Administrator/.android/lint/lint.jar
        jarFiles.addAll(client.findGlobalRuleJars());

        if (!jarFiles.isEmpty()) {
            List<IssueRegistry> registries = Lists.newArrayListWithExpectedSize(jarFiles.size());
            registries.add(registry);
            for (File jarFile : jarFiles) {
                try {
					//从lint.jar获取JarFileIssueRegistry
                    JarFileIssueRegistry registry = JarFileIssueRegistry.get(client, jarFile);
                    if (registry.hasLegacyDetectors()) {
                        runCompatChecks = true;
                    }
                    if (myCustomIssues == null) {
                        myCustomIssues = Sets.newHashSet();
                    }
                    myCustomIssues.addAll(registry.getIssues());
                    registries.add(registry);
                } catch (Throwable e) {
                    client.log(e, "Could not load custom rule jar file %1$s", jarFile);
                }
            }
            if (registries.size() > 1) { // the first item is registry itself
                registry = new CompositeIssueRegistry(registries);
            }
        }
    }
```

demo中为jar地址
E:\github\LintRulesForAndroid\module_news\build\intermediates\bundles\default\lint.jar
### 获得全局lint.jar源码解析
client.findGlobalRuleJars()
这个方法有两种方式获取 全局lint.jar路径，一种是从路径C:/Users/Administrator/.android/lint/lint.jar获取
另外一种从系统环境变量ANDROID_LINT_JARS中获取。
```java
  public List<File> findGlobalRuleJars() {
        // Look for additional detectors registered by the user, via
        // (1) an environment variable (useful for build servers etc), and
        // (2) via jar files in the .android/lint directory
        List<File> files = null;
		//获得androihome下的lintjar
		//我电脑上为C:/Users/Administrator/.android/lint/lint.jar
        try {
            String androidHome = AndroidLocation.getFolder();
            File lint = new File(androidHome + File.separator + "lint");
            if (lint.exists()) {
                File[] list = lint.listFiles();
                if (list != null) {
                    for (File jarFile : list) {
                        if (endsWith(jarFile.getName(), DOT_JAR)) {
                            if (files == null) {
                                files = new ArrayList<>();
                            }
                            files.add(jarFile);
                        }
                    }
                }
            }
        } catch (AndroidLocation.AndroidLocationException e) {
            // Ignore -- no android dir, so no rules to load.
        }
		//从环境变量中获取lint.jar的路径
        String lintClassPath = System.getenv("ANDROID_LINT_JARS");
        if (lintClassPath != null && !lintClassPath.isEmpty()) {
            String[] paths = lintClassPath.split(File.pathSeparator);
            for (String path : paths) {
                File jarFile = new File(path);
                if (jarFile.exists()) {
                    if (files == null) {
                        files = new ArrayList<>();
                    } else if (files.contains(jarFile)) {
                        continue;
                    }
                    files.add(jarFile);
                }
            }
        }

        return files != null ? files : Collections.emptyList();
    }
```
### 根据所有的lint.jar获得JarFileIssueRegistry对象
该对象保存所有的IssueRegistry注册的所有Issues

获取流程大致，是通过Mainfest找到属性值Lint-Registry-v2的变量
如果为空，则继续根据Lint-Registry这个key继续查找，找到注册的value之后反射生成IssueRegistry对象，然后将所有的ISSues保存在JarFileIssueRegistry的issues这个集合里，最终将系统定义和用户定义ISsue统一放在CompositeIssueRegistry中。至此获得所有Issue的流程已完成。

所有的ISsue已经保存在CompositeIssueRegistry，下一步根据Issue获得定义所有Detectors
### 获取所有Detecotr
我们自定义检查都要继承Detector这个类，获取的地方是在analysis()中调用的computeDetectors()

```
    private void computeDetectors(@NonNull Project project) {
        //....
        Configuration configuration = project.getConfiguration(this);
        scopeDetectors = new EnumMap<>(Scope.class);
        applicableDetectors = registry.createDetectors(client, configuration,
                scope, scopeDetectors);


        //....
    }
```
![lintOptions](./pic/lintxml.png)


最终调用IssueRegistry的createDetectors(()方法

#### checkProject检查当前项目和所有依赖项目
```
  private void checkProject(@NonNull Project project, @NonNull Project main) {
     	//.....

        for (Detector check : applicableDetectors) {
//回调Detector的beforeCheckProject方法
            check.beforeCheckProject(projectContext);
            if (canceled) {
                return;
            }
        }

       //.....
		//检查当前项目的java xml class gradle property等等
        runFileDetectors(project, main);

        if (!Scope.checkSingleFile(scope)) {
            List<Project> libraries = project.getAllLibraries();
            for (Project library : libraries) {
                Context libraryContext = new Context(this, library, project, projectDir);
                currentProject = library;

                for (Detector check : applicableDetectors) {
//回调Detector的beforeCheckLibraryProject方法
                    check.beforeCheckLibraryProject(libraryContext);
                    if (canceled) {
                        return;
                    }
                }
                //.....

                runFileDetectors(library, main);
             

               //.....

                for (Detector check : applicableDetectors) {
     //回调Detector的afterCheckLibraryProject方法
                    check.afterCheckLibraryProject(libraryContext);
                    if (canceled) {
                        return;
                    }
                }
            }
        }

        currentProject = project;

        for (Detector check : applicableDetectors) {
        //回调Detector的afterCheckProject方法
            check.afterCheckProject(projectContext);
            if (canceled) {
                return;
            }
        }

     
    }

```
其中一个关键方法,按找一下顺序检查ManifestFiles->xml->java->class->gradle->其他->PROGUARD_FILE->->property file

一个关键方法getSubset，增量检测只检测部分文件或者文件夹的时候会有用。

```
private void runFileDetectors(@NonNull Project project, @Nullable Project main) {
        // Look up manifest information (but not for library projects)
        if (project.isAndroidProject()) {
            for (File manifestFile : project.getManifestFiles()) {
                XmlParser parser = client.getXmlParser();
                if (parser != null) {
                    XmlContext context = new XmlContext(this, project, main, manifestFile, null,
                            parser);
                    context.document = parser.parseXml(context);
                    if (context.document != null) {
                        try {
                            project.readManifest(context.document);

                            if ((!project.isLibrary() || (main != null
                                    && main.isMergingManifests()))
                                    && scope.contains(Scope.MANIFEST)) {
                                List<Detector> detectors = scopeDetectors.get(Scope.MANIFEST);
                                if (detectors != null) {
                                    ResourceVisitor v = new ResourceVisitor(parser, detectors,
                                            null);
                                    fireEvent(EventType.SCANNING_FILE, context);
									//检查mainfest.xml
                                    v.visitFile(context, manifestFile);
                                }
                            }
                        } finally {
                          if (context.document != null) { // else: freed by XmlVisitor above
                              parser.dispose(context, context.document);
                          }
                        }
                    }
                }
            }

            // Process both Scope.RESOURCE_FILE and Scope.ALL_RESOURCE_FILES detectors together
            // in a single pass through the resource directories.
            if (scope.contains(Scope.ALL_RESOURCE_FILES)
                    || scope.contains(Scope.RESOURCE_FILE)
                    || scope.contains(Scope.RESOURCE_FOLDER)
                    || scope.contains(Scope.BINARY_RESOURCE_FILE)) {
                List<Detector> dirChecks = scopeDetectors.get(Scope.RESOURCE_FOLDER);
                List<Detector> binaryChecks = scopeDetectors.get(Scope.BINARY_RESOURCE_FILE);
                List<Detector> checks = union(scopeDetectors.get(Scope.RESOURCE_FILE),
                        scopeDetectors.get(Scope.ALL_RESOURCE_FILES));
                boolean haveXmlChecks = checks != null && !checks.isEmpty();
                List<ResourceXmlDetector> xmlDetectors;
                if (haveXmlChecks) {
                    xmlDetectors = new ArrayList<>(checks.size());
                    for (Detector detector : checks) {
                        if (detector instanceof ResourceXmlDetector) {
                            xmlDetectors.add((ResourceXmlDetector) detector);
                        }
                    }
                    haveXmlChecks = !xmlDetectors.isEmpty();
                } else {
                    xmlDetectors = Collections.emptyList();
                }
                if (haveXmlChecks
                        || dirChecks != null && !dirChecks.isEmpty()
                        || binaryChecks != null && !binaryChecks.isEmpty()) {
                    List<File> files = project.getSubset();
                    if (files != null) {
                        checkIndividualResources(project, main, xmlDetectors, dirChecks,
                                binaryChecks, files);
                    } else {
                        List<File> resourceFolders = project.getResourceFolders();
                        if (!resourceFolders.isEmpty()) {
                            for (File res : resourceFolders) {

						//检查文件夹、xml、二进制文件如png jpg webp

                                checkResFolder(project, main, res, xmlDetectors, dirChecks,
                                        binaryChecks);
                            }
                        }
                    }
                }
            }

            if (canceled) {
                return;
            }
        }

        if (scope.contains(Scope.JAVA_FILE) || scope.contains(Scope.ALL_JAVA_FILES)) {
            List<Detector> checks = union(scopeDetectors.get(Scope.JAVA_FILE),
                    scopeDetectors.get(Scope.ALL_JAVA_FILES));
            if (checks != null && !checks.isEmpty()) {
                List<File> files = project.getSubset();
                if (files != null) {
                    checkIndividualJavaFiles(project, main, checks, files);
                } else {
                    List<File> sourceFolders = project.getJavaSourceFolders();
                    List<File> testFolders = scope.contains(Scope.TEST_SOURCES)
                            ? project.getTestSourceFolders() : Collections.emptyList();
                    checkJava(project, main, sourceFolders, testFolders, checks);
                }
            }
        }

        if (canceled) {
            return;
        }

        if (scope.contains(Scope.CLASS_FILE)
                || scope.contains(Scope.ALL_CLASS_FILES)
                || scope.contains(Scope.JAVA_LIBRARIES)) {
            checkClasses(project, main);
        }

        if (canceled) {
            return;
        }

        if (scope.contains(Scope.GRADLE_FILE)) {
            checkBuildScripts(project, main);
        }

        if (canceled) {
            return;
        }

        if (scope.contains(Scope.OTHER)) {
            List<Detector> checks = scopeDetectors.get(Scope.OTHER);
            if (checks != null) {
                OtherFileVisitor visitor = new OtherFileVisitor(checks);
                visitor.scan(this, project, main);
            }
        }

        if (canceled) {
            return;
        }

        if (project == main && scope.contains(Scope.PROGUARD_FILE) &&
                project.isAndroidProject()) {
            checkProGuard(project, main);
        }

        if (project == main && scope.contains(Scope.PROPERTY_FILE)) {
            checkProperties(project, main);
        }
    }
```

### 打印和输出报告

回到LintCliClient.run(@NonNull IssueRegistry registry, @NonNull List<File> files)方法中
在driver.analyze(createLintRequest(files))之后，有如下代码

```
      Collections.sort(warnings);
	  ...
      boolean hasConsoleOutput = false;
        for (Reporter reporter : flags.getReporters()) {
            reporter.write(stats, warnings);
            if (reporter instanceof TextReporter && ((TextReporter)reporter).isWriteToConsole()) {
                hasConsoleOutput = true;
            }
        }

        if (!flags.isQuiet() && !hasConsoleOutput) {
            System.out.print(String.format("Lint found %1$s",
                    LintUtils.describeCounts(errorCount, warningCount, true)));
            if (baselineErrorCount > 0 || baselineWarningCount > 0) {
                System.out.print(String.format(" (%1$s filtered by baseline %2$s)",
                        LintUtils.describeCounts(stats.baselineErrorCount,
                                stats.baselineWarningCount, true),
                        flags.getBaselineFile().getName()));
            }
            System.out.println();
        }

```
driver将扫描的问题全部保存在warnings这个列表里,然后进行排序输出，输出方式有html，xml和控制台输出
到此基本上整个流程已分析完成

### 如何debug Lint源码
先找的Edit Configurations,
![edit configuratiosns](./pic/edit_configuratiosn.png)
打开点加号添加一个remote类型
![remote](./pic/remote.png)
切换到这个类型
在terminal 输入命令 参数要加上 `-Dorg.gradle.daemon=false -Dorg.gradle.debug=true`
如`gradle app:lintDebug -Dorg.gradle.daemon=false -Dorg.gradle.debug=true`
此时运行起来会一直等待，点击debug按钮
![debug](./pic/debug_unnamed.png)
在点击terminal窗口就跑起来了
![teminal](./pic/sudio_teminal.png)



