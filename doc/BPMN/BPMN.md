# BPMN 2.0 

## 介绍

### 什么是BPMN 2.0

BPMN是一个被广泛接受与支持的，展现流程的注记方法，他是一个行业标准

### 定义流程

BPMN 2.0 流程定义文件（后面统称流程定义文件）以 `.bpmn20.xml` 或 `.bpmn` 为文件后缀

流程定义文件的根元素是 `definitions` ，在根元素内可以定义多个流程定义（对应 `process` 元素），但是为了方便以后的维护，官方建议一个流程定义文件里只定义一个流程定义。

一个空的流程定义如下所示：

```xml
<definitions
  xmlns="http://www.omg.org/spec/BPMN/20100524/MODEL"
  xmlns:flowable="http://flowable.org/bpmn"
  targetNamespace="Examples">

  <process id="myProcess" name="My First Process">
    ..
  </process>

</definitions>
```

注意 `definitions` 元素至少需要包含 `xmlns` 与 `targetNamespace` 声明，但是 `targetNamespace` 允许为空。关于 `targetNamespace` 的作用，查看[部署](部署.md#类别)部分内容。

这里介绍表示流程定义的 `process` 元素的两个属性：

* id 必填，对应 `ProcessDefinition` 对象实例的 `key` 属性。此id属性通常还会被用来启动流程实例 —— `ProcessInstance processInstance = runtimeService.startProcessInstanceByKey("myProcess");` 。前面的方法每次调用时都取最近的部署版本的流程定义。注意 `startProcessInstanceByKey` 方法与 `startProcessInstanceById` 方法不同，后者接受的是部署时产生的[id](部署.md#流程定义的版本)，并且长度也有限制（64个字符）。
* name 选填，[将映射为 `ProcessDefinition` 对象实例的 `name` 属性](部署.md#流程定义的版本)。引擎本身不会使用这个参数，但它可以提升用户交互性（比如显示更容易让人理解的名字）。


### 示例

###### 前置准备

本地独立的h2服务器，并修改数据库连接url为 `jdbc.url=jdbc:h2:tcp://localhost/flowable` 

###### 用例

有一个公司，叫做BPMCorp。在BPMCorp中，由会计部门负责，每月需要为股东撰写一份报告。在报告完成后，需要高层经理中的一人进行审核，然后才能发给所有投资人。

###### 流程图

![financial.report.example.diagram.png](../../img/BPMN/financial.report.example.diagram.png)

如上所示，流程图由启动事件、撰写月度财务报告的用户任务、审核月度财务报告的用户任务和结束事件组成。

###### XML格式

```xml
<definitions id="definitions"
  targetNamespace="http://flowable.org/bpmn20"
  xmlns:flowable="http://flowable.org/bpmn"
  xmlns="http://www.omg.org/spec/BPMN/20100524/MODEL">

    <process id="financialReport" name="Monthly financial report reminder process">

      <startEvent id="theStart" />

      <sequenceFlow id="flow1" sourceRef="theStart" targetRef="writeReportTask" />

      <userTask id="writeReportTask" name="Write monthly financial report" >
        <documentation>
          Write monthly financial report for publication to shareholders.
        </documentation>
        <potentialOwner>
          <resourceAssignmentExpression>
            <formalExpression>accountancy</formalExpression>
          </resourceAssignmentExpression>
        </potentialOwner>
      </userTask>

      <sequenceFlow id="flow2" sourceRef="writeReportTask" targetRef="verifyReportTask" />

      <userTask id="verifyReportTask" name="Verify monthly financial report" >
        <documentation>
          Verify monthly financial report composed by the accountancy department.
          This financial report is going to be sent to all the company shareholders.
        </documentation>
        <potentialOwner>
          <resourceAssignmentExpression>
            <formalExpression>management</formalExpression>
          </resourceAssignmentExpression>
        </potentialOwner>
      </userTask>

      <sequenceFlow id="flow3" sourceRef="verifyReportTask" targetRef="theEnd" />

      <endEvent id="theEnd" />

    </process>

</definitions>
```

上述xml代码中，启动事件对应 `startEvent` 元素，用户任务对应 `userTask` 元素，结束事件对应 `endEvent` 元素。每个元素之间通过 `sequenceFlow` (顺序流) 元素连接，并且用顺序流元素的 `sourceRef` 属性和 `targetRef` 属性指定流向。其中用户任务被分别分配给了 `accountancy` 和 `management` 组。// TODO: 补充任务分配部分链接

###### 启动流程实例

在这个例子中，一个流程实例将对应某一月份的财经报告创建与审核工作，所有月份的流程实例共享相同的流程定义。

启动流程实例前，我们需要先部署流程定义：

```java
Deployment deployment = repositoryService.createDeployment()
                .addClasspathResource("FinancialReportProcess.bpmn20.xml")
                .deploy();
```

部署完流程定义就可以启动流程实例了：

```java
ProcessInstance processInstance = runtimeService.startProcessInstanceByKey("financialReport");
```

流程实例启动后到用户任务处会停止，因为用户任务需要人工处理，这被叫做等待状态。flowable的[事务特性](开始.md#题外话，事务)和[持久化特性](Flowable-API.md#瞬时变量)会与等待状态有关。

###### 任务列表

