## Reading 10

1. [Will Kubernetes Sink the Hadoop Ship?](https://thenewstack.io/will-kubernetes-sink-the-hadoop-ship/)

2. [Things that Data Engineers Must Learn to Exceed the Post-Hadoop Era](https://kyligence.io/blog/things-that-data-engineers-must-learn-to-exceed-the-post-hadoop-era)

3. [Why data scientists love Kubernetes](https://opensource.com/article/19/1/why-data-scientists-love-kubernetes)

4. [Service Mesh - The New Battleground For The Platform Wars](https://www.forbes.com/sites/janakirammsv/2020/09/20/service-meshthe-new-battleground-for-the-platform-wars/?sh=3c1689523021)

5. [Cloud computing storms a bastion of the enterprise: the data warehouse](https://siliconangle.com/2020/11/15/cloud-computing-storms-bastion-enterprise-data-warehouse/)


## Key points

This week's reading is a collection of "state of the world" articles that brings you up-to-date with the current state of the ("Big Data") world.

Article 3 includes pointer to some projects that your project team may wish to pursue including:

* [Seldon](https://github.com/SeldonIO/seldon-core): converts an ML model into a REST microservice
* [Binder](https://github.com/jupyterhub/binderhub): server Jupyter notebooks from a Kubernetes cluster

Article 4 provides a broader picture around [istio](https://istio.io/). Adding a service mesh to your microservices system makes it easy/possible to perform:
* traffic management (circuit breaking, fault injection, traffic shifting, etc)
* blue/green deployment
* canary deployment

[Guide 3](https://scp756-221.github.io/course-site//#/g3-mesh/page?embedded=true&hidegitlink=true) walks you through a few of these.

Article 5 dates from November 2020 which is only 16 months ago. The debate between data warehouse vs data lake has already evolved towards a so-called data lakehouse ("an extended data lake") that provides data warehousing capability. See [Dremio](https://www.dremio.com/platform/cloud/), [Databrick's Delta Lake](https://databricks.com/product/delta-lake-on-databricks), [Tabular](https://tabular.io/), and likely others.


## Parts to skim

Article 1 & 2 are decidedly pitching a product; you can safely ignore them (Iguazio in the former and Kyligence in the latter).

## Your Turn

   Take 30 minutes for [Quiz 10](https://coursys.sfu.ca/2022sp-cmpt-756-g1/+q10/). 

