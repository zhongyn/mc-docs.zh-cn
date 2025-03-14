---
title: 教程：设置 Azure 时序见解预览版环境 | Microsoft Docs
description: 了解如何在 Azure 时序见解预览中设置环境。
author: ashannon7
ms.author: anshan
ms.workload: big-data
manager: cshankar
ms.service: time-series-insights
services: time-series-insights
ms.topic: tutorial
origin.date: 06/18/2019
ms.date: 07/08/2019
ms.custom: seodec18
ms.openlocfilehash: 9bdbb56b6c94f083f6c03a2790b2992f6e4145ea
ms.sourcegitcommit: c0f7c439184efa26597e97e5431500a2a43c81a5
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/28/2019
ms.locfileid: "67456430"
---
# <a name="tutorial-set-up-an-azure-time-series-insights-preview-environment"></a>教程：设置 Azure 时序见解预览环境

本教程将逐步引导你创建一个 Azure 时序见解预览版即付即用 (PAYG) 环境。 

本教程介绍如何执行下列操作：

* 创建 Azure 时序见解预览环境。
* 将 Azure 时序见解预览环境连接到“Azure 事件中心”中的事件中心。
* 运行解决方案加速器示例，以将数据流式传输到 Azure 时序见解预览版环境。
* 对数据进行基本的分析。
* 定义时序模型类型和层次结构，并将其与实例相关联。

>[!TIP]
> [IoT 解决方案加速器](https://www.azureiotsolutions.com/Accelerators)提供企业级的预配置解决方案，可用于加速自定义 IoT 解决方案的开发。

## <a name="create-a-device-simulation"></a>创建设备模拟

在本部分，我们将创建三个模拟设备，用于将数据发送到 Azure IoT 中心实例。

1. 转到 [Azure IoT 解决方案加速器页](https://www.azureiotsolutions.com/Accelerators)。 该页显示了多个预生成的示例。 使用 Azure 帐户登录。 然后，选择“设备模拟”  。

   [![Azure IoT 解决方案加速器页](media/v2-update-provision/device-one-accelerator.png)](media/v2-update-provision/device-one-accelerator.png#lightbox)

   选择“立即试用”。 

1. 在“创建设备模拟解决方案”  页上，设置以下参数：

    | 参数 | 操作 |
    | --- | --- |
    | **部署名称** | 输入用于新资源组的唯一值。 将会创建列出的 Azure 资源并将其分配到资源组。 |
    | **Azure 订阅** | 选择曾经用于创建时序见解环境的订阅。 |
    | **Azure 位置** | 选择曾经用于创建时序见解环境的区域。 |
    | **部署选项** | 选择“预配新的 IoT 中心”。  |
 
    选择“创建解决方案”  。 部署完该解决方案最长可能需要 20 分钟。

    [![“创建设备模拟解决方案”页](media/v2-update-provision/device-two-create.png)](media/v2-update-provision/device-two-create.png#lightbox)

## <a name="create-a-time-series-insights-preview-payg-environment"></a>创建时序见解预览 PAYG 环境

本部分介绍如何使用 [Azure 门户](https://portal.azure.cn/)创建 Azure 时序见解预览版环境，并将其连接到 IoT 解决方案加速器创建的 IoT 中心。

1. 使用 Azure 订阅帐户登录到 Azure 门户。

1. 选择“创建资源” > “物联网” > “时序见解”    。

   [![依次选择“物联网”、“时序见解”](media/v2-update-provision/payg-one-azure.png)](media/v2-update-provision/payg-one-azure.png#lightbox)

1. 在“创建时序见解环境”  窗格的“基本信息”  选项卡上，设置以下参数：

    | 参数 | 操作 |
    | --- | ---|
    | **环境名称** | 输入 Azure 时序见解预览版环境的唯一名称。 |
    | **订阅** | 输入想要在其中创建 Azure 时序见解预览版环境的订阅。 最佳做法是使用与设备模拟器创建的其他 IoT 资源相同的订阅。 |
    | **资源组** | 为 Azure 时序见解预览版环境资源选择现有的资源组或创建新的资源组。 资源组是 Azure 资源的容器。 最佳做法是使用与设备模拟器创建的其他 IoT 资源相同的资源组。 |
    | **Location** | 为 Azure 时序见解预览版环境选择数据中心区域。 为了避免延迟增大，最好是在其他 IoT 资源所在的区域创建 Azure 时序见解预览版环境。 |
    | **层** |  选择“PAYG”（即用即付）。   这是 Azure 时序见解预览版产品的 SKU。 |
    | **属性 ID** | 输入一个用于唯一标识时序实例的值。 在“属性 ID”框中输入的值  是固定不变的。 之后不能更改它。 在本教程中，请输入 **iothub-connection-device-id**。若要详细了解时序 ID，请参阅[选择时序 ID 的最佳做法](./time-series-insights-update-how-to-id.md)。 |
    | **存储帐户名称** | 为要创建的新存储帐户输入全局唯一名称。 |
   
   在完成时选择“下一步:**事件源”** 。

   [![用于创建时序见解环境的窗格](media/v2-update-provision/payg-two-create.png)](media/v2-update-provision/payg-two-create.png#lightbox)

1. 在“事件源”选项卡上，设置以下参数： 

   | 参数 | 操作 |
   | --- | --- |
   | **创建事件源?** | 请选择“是”。 |
   | **名称** | 输入事件源名称的唯一值。 |
   | **源类型** | 选择“IoT 中心”  。 |
   | **选择中心** | 选择“选择现有”。  |
   | **订阅** | 选择曾经用于设备模拟器的订阅。 |
   | **IoT 中心名称** | 选择为设备模拟器创建的 IoT 中心名称。 |
   | **Iot 中心访问策略** | 选择“iothubowner”。  |
   | **IoT 中心使用者组** | 选择“新建”，输入唯一名称，然后选择“添加”。   在 Azure 时序见解预览版中，使用者组必须是唯一值。 |
   | **时间戳属性** | 此值用于标识传入遥测数据中的**时间戳**属性。 在本教程中，请将此框留空。 此模拟器使用 IoT 中心的传入时间戳，时序见解默认使用该时间戳。 |

   选择“查看 + 创建”  。

   [![配置事件源](media/v2-update-provision/payg-five-event-source.png)](media/v2-update-provision/payg-five-event-source.png#lightbox)

1. 在“查看 + 创建”选项卡上查看所做的选择，然后选择“创建”。  

    [![包含“创建”按钮的“查看 + 创建”页](media/v2-update-provision/payg-six-review.png)](media/v2-update-provision/payg-six-review.png#lightbox)

    可以看到部署状态：

    [![指出部署已完成的通知](media/v2-update-provision/payg-seven-deploy.png)](media/v2-update-provision/payg-seven-deploy.png#lightbox)

1. 如果你是租户所有者，则可访问自己的 Azure 时序见解预览版环境。 确保具有以下访问权限：

   1. 搜索资源组，然后选择自己的 Azure 时序见解预览版环境：

      [![所选环境](media/v2-update-provision/payg-eight-environment.png)](media/v2-update-provision/payg-eight-environment.png#lightbox)

   1. 在“Azure 时序见解预览版”页上，选择“数据访问策略”  ：

      [![数据访问策略](media/v2-update-provision/payg-nine-data-access.png)](media/v2-update-provision/payg-nine-data-access.png#lightbox)

   1. 验证凭据是否已列出：

      [![列出的凭据](media/v2-update-provision/payg-ten-verify.png)](media/v2-update-provision/payg-ten-verify.png#lightbox)

   如果未列出你的凭据，则必须授予自己访问该环境的权限。 若要详细了解如何设置权限，请参阅[授予数据访问权限](./time-series-insights-data-access.md)。

## <a name="stream-data-into-your-environment"></a>将数据流式传输到环境

1. 导航回到 [Azure IoT 解决方案加速器页](https://www.azureiotsolutions.com/Accelerators)。 在解决方案加速器仪表板中找到你的解决方案。 然后选择“启动”： 

    [![启动设备模拟解决方案](media/v2-update-provision/device-three-launch.png)](media/v2-update-provision/device-three-launch.png#lightbox)

1. 随后你将重定向到“Microsoft Azure IoT 设备模拟”页  。 在页面右上角选择“新建模拟”。 

    [![Azure IoT 模拟页](media/v2-update-provision/device-four-iot-sim-page.png)](media/v2-update-provision/device-four-iot-sim-page.png#lightbox)

1. 在“模拟设置”  窗格中，设置以下参数：

    | 参数 | 操作 |
    | --- | --- |
    | **名称** | 为模拟器输入唯一名称。 |
    | **说明** | 输入定义。 |
    | **模拟持续时间** | 设置为“无限期运行”。  |
    | **设备型号** | **名称**：输入“冷却器”。  <br />**数量**：输入 **3**。 |
    | **目标 IoT 中心** | 设置为“使用预配的 IoT 中心”。  |

    [![要设置的参数](media/v2-update-provision/device-five-params.png)](media/v2-update-provision/device-five-params.png#lightbox)

    选择“开始模拟”  。

    在设备模拟仪表板中，注意针对“活动设备”和“每秒消息数”显示的信息   。

    [![Azure IoT 模拟仪表板](media/v2-update-provision/device-seven-dashboard.png)](media/v2-update-provision/device-seven-dashboard.png#lightbox)

## <a name="analyze-data-in-your-environment"></a>在环境中分析数据

在此部分，使用 [Azure 时序见解预览资源管理器](./time-series-insights-update-explorer.md)对时序数据进行基本的分析。

1. 在 [Azure 门户](https://portal.azure.cn/)中的资源页上选择相应的 URL，转到 Azure 时序见解预览版资源管理器。

    [![时序见解预览版资源管理器 URL](media/v2-update-provision/analyze-one-portal.png)](media/v2-update-provision/analyze-one-portal.png#lightbox)

1. 在资源管理器中，选择“时序见解实例”节点，以查看环境中的所有 Azure 时序见解预览版实例  。

    [![无父级实例的列表](media/v2-update-provision/analyze-two-unparented.png)](media/v2-update-provision/analyze-two-unparented.png#lightbox)

1. 选择第一个时序实例。 然后，选择“显示压强”。 

    [![所选的时序实例，带有的菜单命令可用于显示平均压强](media/v2-update-provision/analyze-three-show-pressure.png)](media/v2-update-provision/analyze-three-show-pressure.png#lightbox)

    此时会显示时序图。 将“时间间隔”更改为“15 秒”。  

    [![时序图](media/v2-update-provision/analyze-four-chart.png)](media/v2-update-provision/analyze-four-chart.png#lightbox)

1. 针对另外两个时序实例重复步骤 3。 可以查看所有时序实例，如以下图表所示：

    [![所有时序的图表](media/v2-update-provision/analyze-five-chart.png)](media/v2-update-provision/analyze-five-chart.png#lightbox)

1. 在“时间范围”选项框中修改时间范围，查看过去一小时的时序趋势： 

    [![将时间范围设置为小时](media/v2-update-provision/analyze-six-time.png)](media/v2-update-provision/analyze-six-time.png#lightbox)

## <a name="define-and-apply-a-model"></a>定义并应用模型

在本部分，你将应用一个模型来构造数据。 若要完成该模型，需要定义类型、层次结构和实例。 若要详细了解数据建模，请参阅[时序模型](./time-series-insights-update-tsm.md)。

1. 在资源管理器中，选择“模型”  选项卡：

   [![资源管理器中的“模型”选项卡](media/v2-update-provision/define-one-model.png)](media/v2-update-provision/define-one-model.png#lightbox)

1. 选择“添加”以添加类型： 

   [![类型的“添加”按钮](media/v2-update-provision/define-two-add.png)](media/v2-update-provision/define-two-add.png#lightbox)

1. 接下来，为类型定义三个变量：压强、温度和湿度。    在“添加类型”  窗格中，设置以下参数：

    | 参数 | 操作 |
    | --- | ---|
    | **名称** | 输入“冷却器”。  |
    | **说明** | 输入“这是冷却器的类型定义”。  |

   * 若要在“变量”下定义“压强”，请设置以下参数：  

     | 参数 | 操作 |
     | --- | ---|
     | **名称** | 输入“平均压力”。  |
     | **值** | 选择“压力(双精度型)”  。 在 Azure 时序见解预览版开始接收事件之后，可能需要等待几分钟才会自动填充“值”。  |
     | **聚合操作** | 选择“AVG”。  |

      [![用于定义压力的选项](media/v2-update-provision/define-three-variable.png)](media/v2-update-provision/define-three-variable.png#lightbox)

      若要添加下一变量，请选择“添加变量”  。

   * 定义温度  ：

     | 参数 | 操作 |
     | --- | ---|
     | **名称** | 输入“平均温度”。  |
     | **值** | 选择“温度(双精度型)”  。 在 Azure 时序见解预览版开始接收事件之后，可能需要等待几分钟才会自动填充“值”。  |
     | **聚合操作** | 选择“AVG”。 |

      [![用于定义温度的选项](media/v2-update-provision/define-four-avg.png)](media/v2-update-provision/define-four-avg.png#lightbox)

      若要添加下一变量，请选择“添加变量”  。

   * 定义湿度： 

      | | |
      | --- | ---|
      | **名称** | 输入“最大湿度”  |
      | **值** | 选择“湿度(双精度型)”  。 在 Azure 时序见解预览版开始接收事件之后，可能需要等待几分钟才会自动填充“值”。  |
      | **聚合操作** | 选择“MAX”。 |

      [![用于定义温度的选项](media/v2-update-provision/define-five-humidity.png)](media/v2-update-provision/define-five-humidity.png#lightbox)

    选择“创建”  。

    可以看到添加的类型：

    [![有关添加类型的信息](media/v2-update-provision/define-six-type.png)](media/v2-update-provision/define-six-type.png#lightbox)

1. 下一步是添加层次结构。 在“层次结构”下选择“添加”：  

    [![包含“添加”按钮的“层次结构”选项卡](media/v2-update-provision/define-seven-hierarchy.png)](media/v2-update-provision/define-seven-hierarchy.png#lightbox)

1. 在“编辑层次结构”  窗格中，设置以下参数：

   | 参数 | 操作 |
   | --- | ---|
   | **名称** | 输入“位置层次结构”。  |
   | **级别 1** | 输入“国家/地区”。  |
   | **级别 2** | 输入“城市”。  |
   | **级别 3** | 输入“建筑物”。  |

   选择“其他安全性验证”  。

    [![包含“创建”按钮的“层次结构”字段](media/v2-update-provision/define-eight-add-hierarchy.png)](media/v2-update-provision/define-eight-add-hierarchy.png#lightbox)

   可以看到创建的层次结构：

    [![有关层次结构的信息](media/v2-update-provision/define-nine-created.png)](media/v2-update-provision/define-nine-created.png#lightbox)

1. 选择“实例”。  选择第一个实例，然后选择“编辑”  ：

    [![选择实例对应的“编辑”按钮](media/v2-update-provision/define-ten-edit.png)](media/v2-update-provision/define-ten-edit.png#lightbox)

1. 在“编辑实例”  窗格中，设置以下参数：

    | 参数 | 操作 |
    | --- | --- |
    | **类型** | 选择“冷却器”。  |
    | **说明** | 输入“冷却器 01.1 的实例”。  |
    | **层次结构** | 选择“位置层次结构”  。 |
    | **国家/地区** | 输入“美国”。  |
    | **城市** | 输入“西雅图”。  |
    | **建筑物** | 输入“太空针塔”。  |

    [![带“保存”按钮的实例字段](media/v2-update-provision/define-eleven-chiller.png)](media/v2-update-provision/define-eleven-chiller.png#lightbox)

   选择“其他安全性验证”  。

1. 针对其他传感器重复上一步骤。 更新以下值：

   * 对于冷却器 01.2：

     | 参数 | 操作 |
     | --- | --- |
     | **类型** | 选择“冷却器”。  |
     | **说明** | 输入“冷却器 01.2 的实例”。  |
     | **层次结构** | 选择“位置层次结构”  。 |
     | **国家/地区** | 输入“美国”。  |
     | **城市** | 输入“西雅图”。  |
     | **建筑物** | 输入“太平洋科学馆”。  |

   * 对于冷却器 01.3：

     | 参数 | 操作 |
     | --- | --- |
     | **类型** | 选择“冷却器”。  |
     | **说明** | 输入“冷却器 01.3 的实例”。  |
     | **层次结构** | 选择“位置层次结构”  。 |
     | **国家/地区** | 输入“美国”。  |
     | **城市** | 输入“纽约”。  |
     | **建筑物** | 输入“帝国大厦”。  |

1. 选择“分析”选项卡，然后刷新页面  。 在“位置层次结构”下展开所有层次结构级别，显示时序实例： 

   [![“分析”选项卡](media/v2-update-provision/define-twelve.png)](media/v2-update-provision/define-twelve.png#lightbox)

1. 若要浏览过去一小时的时序实例，请将“快速选择时间”更改为“过去一小时”：  

    [![“QuickTime”框，其中已选择“过去一小时”](media/v2-update-provision/define-thirteen-explore.png)](media/v2-update-provision/define-thirteen-explore.png#lightbox)

1. 在“太平洋科学中心”下选择时序实例，然后选择“显示最大湿度”   。

    [![选择的时序实例，以及“显示最大湿度”菜单选项](media/v2-update-provision/define-fourteen-show-max.png)](media/v2-update-provision/define-fourteen-show-max.png#lightbox)

1. 此时会打开间隔大小为 1 分钟的“最大湿度”时序   。 若要筛选范围，请选择某个区域。 若要分析时间范围内的事件，请右键单击图表，然后选择“缩放”： 

   [![选择的范围，以及快捷菜单上的“缩放”命令](media/v2-update-provision/define-fifteen-filter.png)](media/v2-update-provision/define-fifteen-filter.png#lightbox)

1. 若要查看事件详细信息，请选择一个区域，然后右键单击图表：

   [![事件的详细列表](media/v2-update-provision/define-eighteen.png)](media/v2-update-provision/define-eighteen.png#lightbox)

## <a name="next-steps"></a>后续步骤

在本教程中，你已学习了如何执行以下操作：  

> [!div class="checklist"]
> * 创建并使用设备模拟加速器。
> * 创建 Azure 时序见解预览 PAYG 环境。
> * 将 Azure 时序见解预览环境连接到事件中心。
> * 运行解决方案加速器示例，以将数据流式传输到 Azure 时序见解预览版环境。
> * 对数据进行基本的分析。
> * 定义时序模型类型和层次结构，并将其与实例相关联。

知道如何创建自己的 Azure 时序见解预览环境后，现在来了解更多关于 Azure 时序见解的关键概念。

了解 Azure 时序见解存储配置：

> [!div class="nextstepaction"]
> [Azure 时序见解预览版存储和入口](./time-series-insights-update-storage-ingress.md)

详细了解时序模型：

> [!div class="nextstepaction"]
> [Azure 时序见解预览数据模型](./time-series-insights-update-tsm.md)

<!-- Images -->
[1]: media/v2-update-provision/device-one-accelerator.png
[2]: media/v2-update-provision/device-two-create.png
[3]: media/v2-update-provision/device-three-launch.png
[4]: media/v2-update-provision/device-four-iot-sim-page.png
[5]: media/v2-update-provision/device-five-params.png
[6]: media/v2-update-provision/device-seven-dashboard.png
[7]: media/v2-update-provision/device-six-listings.png
[8]: media/v2-update-provision/device-eight-portal.png

[9]: media/v2-update-provision/payg-one-azure.png
[10]: media/v2-update-provision/payg-two-create.png
[11]: media/v2-update-provision/payg-three-new.png
[12]: media/v2-update-provision/payg-four-add.png
[13]: media/v2-update-provision/payg-five-event-source.png
[14]: media/v2-update-provision/payg-six-review.png
[15]: media/v2-update-provision/payg-seven-deploy.png
[16]: media/v2-update-provision/payg-eight-environment.png
[17]: media/v2-update-provision/payg-nine-data-access.png
[18]: media/v2-update-provision/payg-ten-verify.png

[19]: media/v2-update-provision/analyze-one-portal.png
[20]: media/v2-update-provision/analyze-two-unparented.png
[21]: media/v2-update-provision/analyze-three-show-pressure.png
[22]: media/v2-update-provision/analyze-four-chart.png
[23]: media/v2-update-provision/analyze-five-chart.png
[24]: media/v2-update-provision/analyze-six-from.png
[25]: media/v2-update-provision/analyze-seven-change-from.png
[26]: media/v2-update-provision/analyze-eight-all.png

[27]: media/v2-update-provision/define-one-model.png
[28]: media/v2-update-provision/define-two-add.png
[29]: media/v2-update-provision/define-three-variable.png
[30]: media/v2-update-provision/define-four-avg.png
[31]: media/v2-update-provision/define-five-humidity.png
[32]: media/v2-update-provision/define-six-type.png
[33]: media/v2-update-provision/define-seven-hierarchy.png
[34]: media/v2-update-provision/define-eight-add-hierarchy.png
[35]: media/v2-update-provision/define-nine-created.png
[36]: media/v2-update-provision/define-ten-edit.png
[37]: media/v2-update-provision/define-eleven-chiller.png
[38]: media/v2-update-provision/define-twelve.png
[39]: media/v2-update-provision/define-thirteen-explore.png
[40]: media/v2-update-provision/define-fourteen-show-max.png
[41]: media/v2-update-provision/define-fifteen-filter.png
[42]: media/v2-update-provision/define-sixteen.png
[43]: media/v2-update-provision/define-seventeen.png
[44]: media/v2-update-provision/define-eighteen.png