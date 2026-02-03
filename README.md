# poc_map

## 概述

poc_map是一款用于nemoV3里，根据web指纹自动匹配poc的工具。在NemoV3的任务流程里，指纹获取与poc扫描是两个独立的子任务。目前Poc扫描主要是通过调用nuclei及相应的poc文件，指纹信息则是通过调用httpx获取基本的web信息，通过加载chainreactor/fingers和fingerprinthub指纹库被动匹配指纹。

为了提高poc扫描的准确性和效率，最佳的实践方案是根据获取的指纹信息，结合已有的poc库，精准匹配后进行poc扫描。因此，我们设计了poc_map模块，通过对已有的poc库进行分类，并结合指纹信息，对poc进行匹配，提高poc扫描的准确性和效率。

## 功能设计

1、根据已有的poc库进行分类、梳理出实战中最常见的poc。目前可以参考的poc库有：

- [nuclei-templates](https://github.com/projectdiscovery/nuclei-templates)
- [chainreactors-templates-neutron](https://github.com/chainreactors/templates/tree/master/neutron)
- [fscan](https://github.com/shadow1ng/fscan/tree/dev/WebScan/pocs)

2、根据梳理出的实战中最常见的poc，从指纹库中提取指纹名称，用以匹配poc。指纹库目前使用两个公开来源、一个私有来源：

- [chainreactors-templates-fingers](https://github.com/chainreactors/templates/tree/master/fingers/http)
- [fingerprinthub](https://github.com/0x727/FingerprintHub)
- private_nuclei_templates(私有库)

3、生成poc_map.json文件，记录poc的分类、匹配指纹、匹配poc文件路径等信息。

## 架构设计

### 1、poc_map模块架构

```bash
├── poc_map
│   ├── web_poc_map_v2.yaml #yaml定义文件
│   ├── web_poc_map_v2.json # 最终生成的poc_map文件
│   ├── web_poc_map_v2.csv  # csv格式的简要汇总表，用于展示和快速查询
│   ├── main.go             # 主函数，对整个poc_map进行校验，并生成web_poc_map_v2.json文件

```

### 2、字段定义

- name：指纹及poc对应的名称，用于标识指纹及poc的对应关系；字符串类型。
- category：指纹对应的poc分类，用于标识poc的分类，如cdn、cms、framework等；字符串类型。
- updated：更新时间。
- description：poc的描述信息，用于说明poc的功能、漏洞等；可选；字符串类型。
- fingerprint：指纹名称，用于与nemo中获取的指纹进行匹配；字符串类型的数组，比如["solr","solr8.x"]等，数组元素可以是多个，只要nemo中的指纹名称中包含任一元素，就认为匹配成功。
- poc：结构体数组，用于存放匹配到的poc文件路径；结构体中包含两个字段：source、path；source表示poc来源，为常量定义（见下方）、path表示对应的poc库中的文件路径名。

### 3、常量定义

#### 1、source

- 1：[nuclei-templates](https://github.com/projectdiscovery/nuclei-templates)，nuclei官方模板库；
- 2：[some_nuclei_templates](https://github.com/hanc00l/some_nuclei_templates)，hanc00l维护的模板库（目前还比较少，后续陆续补充）；
- 3：private_nuclei_templates，私有poc库。

### 4、yaml文件示例

```yaml
- name: finereport
  category: component
  description: finereport,帆软报表系统
  updated: "2025-05-06"
  fingerprint:
    - finereport
  poc:
    - source: some_nuclei_templates
      path:
        - finereport_data_decision_system_unserialize.yaml
    - source: nuclei-template
      path:
        - http/vulnerabilities/finereport/fine-report-v9-file-upload.yaml
        - http/vulnerabilities/finereport/finereport-path-traversal.yaml
        - http/vulnerabilities/finereport/finereport-sqli-rce.yaml
```

## 其他

感谢上述的poc库作者，他们的贡献是本项目的源动力。