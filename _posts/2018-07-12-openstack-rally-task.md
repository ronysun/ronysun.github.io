---
layout: post  
title: rally task start命令代码及task配置文件分析   
categories:  
  - openstack  
---
## rally task配置文件
Rally本身提供了一些task配置文件，用于提供测试用例中所需的信息包括场景测试中所传入的参数、运行方式是并行还是串行，context等信息。Rally本身提供的task 配置文件在rally/samples/task/scenarios/目录下。

```yaml
---
  NovaServers.boot_and_delete_server:
    -
      args:
        flavor:
            name: "{{flavor_name}}"
        image:
            name: "^cirros.*-disk$"
        force_delete: false
      runner:
        type: "constant"
        times: 10
        concurrency: 2
      context:
        users:
          tenants: 3
          users_per_tenant: 2
      sla:
        failure_rate:
          max: 0
    -
      args:
        flavor:
            name: "{{flavor_name}}"
        image:
            name: "^cirros.*-disk$"
        auto_assign_nic: true
      runner:
        type: "constant"
        times: 10
        concurrency: 2
      context:
        users:
          tenants: 3
          users_per_tenant: 2
        network:
          start_cidr: "10.2.0.0/24"
          networks_per_tenant: 2
      sla:
        failure_rate:
          max: 0
```
其中，args提供测试用例使用的类型方法的参数，context提供运行测试用例的上下文环境变量信息，包括模拟几个用户同时测试、租户情况等；quotas指定测试中涉及到资源的配额限制，runner提供测试的运行器情况，比如，并发测试、串行测试，等等。  
rally目前支持4种runner类型，通过task配置文件中runner参数进行区分，包括：constant、constant_for_duration、rps、serial。
1. constant方式，是利用multiprocessing的Pool方式创建一个进程池，池中的进程数量等于runner的配置参数concurrency，执行每个task时，由池中的所有进程同时测试、模拟多用户并发访问的情况。constant方式中要求提供参数times，用于指定一个task中执行测试用例的次数。
1. constant_for_duration方式，与constant相似，也是构建一个进程池实现并发测试，区别在于，constant_for_duration方式要求额外提供一个参数duration，但是没有参数times。该参数用于指明执行测试的时间， rally一直执行task指定的测试用例，直到达到duration规定的时间长度，任务结束；
1. rps方式，测试任务平均分配到每个processer上，通过对每个process创建thread实现高并发测试。不同于前两种方式（使用multiprocessing.Pool 构建进程池），rps方式使用multiprocessing.Process构建执行task的worker，每个worker执行测试次数的总和是配置中的times，每个worker的rps总和是配置中rps。workers的数量由times和运行rally测试的主机上实际processer数量的最小值确定，times平均分配到每个worker上
1. serial方式，是使用一个process串行的执行测试。

## rally task-start命令执行流程

## rally脚本分析
Rally完成每个场景的测试是通过相应client调用openstack API操作。
以下举例分析场景create_and_list_networks函数。在执行函数之前会有一些validation装饰器。
1. **validation.required_openstack**   
 验证scenario类型方法运行时对于用户角色（admin/user）的要求是否得到满足，如果要求admin用户执行task，则在装饰器参数中提供admin=True，如果要求普通用户身份执行，提供users=True，默认两个值都是False。使用此装饰器，上述两个参数至少提供一个为True。检验过程中会查询当前deployment的配置，是否提供scenario要求的用户角色。
1. **validation.required_services**   
检验执行scenario类型方法测试时，要求的openstack service是否可用，使用装饰器时，要求在参数中提供待检验的Service名称，并且名称的书写必须包含在constants._Service所定义服务的范围内；此外，检验时，通过openstackclient查询services列表，待检查的服务应在services列表中；否则验证失败。
1. **scenario.configure**   
用于设置待测试方法的is_scenario和context属性，分别用于支持task的validate过程，以及通过在context中增加cleanup参数，实现测试后清理环境。
```python
@validation.required_services(consts.Service.NEUTRON)
@validation.required_openstack(users=True)
@scenario.configure(context={"cleanup": ["neutron"]},
                    name="NeutronNetworks.create_and_list_networks")
class CreateAndListNetworks(utils.NeutronScenario):

    def run(self, network_create_args=None):
        """Create a network and then list all networks.

        Measure the "neutron net-list" command performance.

        If you have only 1 user in your context, you will
        add 1 network on every iteration. So you will have more
        and more networks and will be able to measure the
        performance of the "neutron net-list" command depending on
        the number of networks owned by users.

        :param network_create_args: dict, POST /v2.0/networks request options
        """
        self._create_network(network_create_args or {})
        self._list_networks()
```
[原文](https://blog.csdn.net/bc_vnetwork/article/details/51818514)
