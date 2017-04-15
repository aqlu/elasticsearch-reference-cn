# 安装Elasticsearch

Elasticsearch提供了以下安装包格式：

格式        |  说明
-----------|-------------
zip/tar.gz |`zip`和`tar.gz`的安装包适合在任何系统上安装，是开始使用Elasticsearch的最简单的选择。<br/> [.zip或.tar.gz文件的安装方式](./Installing_Elasticsearch/Install_Elasticsearch_with_.zip_or_.tar.gz.md) 或 [Windows安装方式](./Installing_Elasticsearch/Install_Elasticsearch_on_Windows.md)
deb        |`deb`安装包适用于Debian，Ubuntu和其他基于Debian的系统。 Debian软件包可以从Elasticsearch网站或我们的Debian仓库下载。<br/>[Debian软件包安装方式](./Installing_Elasticsearch/Install_Elasticsearch_with_Debian_Package.md)
rpm        |`rpm`包适合在Red Hat，Centos，SLES，OpenSuSE和其他基于RPM的系统上安装。 RPM可以从Elasticsearch网站或从我们的RPM仓库下载。[rpm软件包安装方式](./Installing_Elasticsearch/Install_Elasticsearch_with_RPM.md)。
docker     | 使用镜像可将Elasticsearch在Docker容器运行。它预装了[X-Pack](https://www.elastic.co/guide/en/x-pack/5.3/index.html)，可以从`Elastic Docker Registry`下载。 [Docker安装方式](./Installing_Elasticsearch/Install_Elasticsearch_with_Docker.md)

## 配置管理工具

我们还提供以下配置管理工具来帮助大型部署：

名称        |项目地址
-----------|-------------
Puppet     |[puppet-elasticsearch](https://github.com/elastic/puppet-elasticsearch)
Chef       |[cookbook-elasticsearch](https://github.com/elastic/cookbook-elasticsearch)
Ansible    |[ansible-elasticsearch](https://github.com/elastic/ansible-elasticsearch)
