<properties linkid="manage-services-import-export" urlDisplayName="Azure Import/Export Service" pageTitle="Using import/export to transfer data to Blob Storage | Microsoft Azure" metaKeywords="" description="Learn how to create import and export jobs in the Azure Management Portal to transfer data to blob storage." metaCanonical="" disqusComments="1" umbracoNaviHide="0" title="Using the Azure Import/Export Service to Transfer Data to Blob Storage" authors="tamram" manager="mbaldwin" editor="cgronlun" />

# 使用 Microsoft Azure 导入/导出服务将数据传输到 Blob 存储中

在通过网络上载成本过高或不可行时，你可以使用 Microsoft Azure 导入/导出服务将大量文件数据传输到 Azure Blob 存储中。你还可以使用导入/导出服务将 Blob 存储中驻留的大量数据以即时且经济高效的方式传输到你的本地安装中。

若要将大量文件数据传输到 Blob 存储中，可以将包含这些数据的一个或多个硬盘驱动器运送到 Azure 数据中心，在那里，你的数据将上载到你的存储帐户中。同样，若要从 Blob 存储中导出数据，你可以将空的硬盘驱动器运送到 Azure 数据中心，在那里，来自你的存储帐户的 Blob 数据将被复制到你的硬盘驱动器，然后返还给你。在你运送包含数据的驱动器之前，你将对该驱动器中的数据进行加密；在导入/导出服务导出你的数据以便发送给你时，也将在运送前对数据进行加密。

你可以通过以下两种方式中的一种创建和管理导入和导出作业：

-   通过使用 Azure 管理门户。
-   通过使用服务的 REST 接口。

本文概述了该导入/导出服务并且说明了如何通过管理门户来使用该导入/导出服务。有关 REST API 的信息，请参阅 [Azure 导入/导出服务 REST API 参考][]。

## 导入/导出服务概述

若要开始从 Blob 存储进行导入或导出的过程，你首先要创建一个“作业”**。作业可以是“导入作业”**或“导出作业”**：

-   需要将本地数据传输到 Azure 存储帐户中的 Blob 时，可创建导入作业。
-   在你想要将当前作为 Blob 存储于你的存储帐户中的数据传输到运送给你的硬盘驱动器时创建导出作业。

创建作业时，需通知导入/导出服务：你要将一个或多个硬盘驱动器运送到 Azure 数据中心。对于某一导入作业，你将要运送包含文件数据的硬盘驱动器。导出作业只需运送空的硬盘驱动器。

若要为导入作业准备要运送的驱动器，你需要运行 **Microsoft Azure 导入/导出工具**，该工具可帮助你将数据复制到驱动器、使用 BitLocker 对驱动器上的数据进行加密以及生成驱动器日志文件。我们将在下文中讨论此方面的内容。

**说明**

必须使用 BitLocker 驱动器加密对驱动器上的数据进行加密。这将在运送过程中保护你的数据。对于导出作业，该导入/导出服务在将驱动器运送回你处之前对你的数据进行加密。

在你创建导入作业或导出作业时，还需要“驱动器 ID”**，这是驱动器制造商分配给特定硬盘的序列号。该驱动器 ID 显示在驱动器的表面。

### 要求和范围

1.  **订阅和存储帐户：**你必须已拥有 Azure 订阅以及一个或多个存储帐户，才能使用导入/导出服务。每个作业只能用于将数据传输到一个存储帐户或者从一个存储帐户传输数据。换言之，一个作业不能跨多个存储帐户。有关创建新存储帐户的信息，请参见[如何创建存储帐户][]。
2.  **硬盘驱动器：**只支持对 3.5 英寸 SATA II/III 硬盘驱动器使用导入/导出服务。不支持大于 4TB 的硬盘驱动器。对于导入作业，将处理驱动器上的第一个数据卷。该数据卷必须使用 NTFS 进行格式化。你可以使用 SATA II/III USB 适配器在外部将 SATA II/III 磁盘连接到大多数计算机。
3.  **BitLocker 加密：** 必须使用 BitLocker 通过用数字密码保护的加密密钥对在硬盘驱动器上存储的所有数据进行加密。
4.  **Blob 存储目标：** 可以将数据上载到块 Blob 和页 Blob 或者从块 Blob 和页 Blob 下载数据。
5.  **作业数目：** 对于每个存储帐户，一个客户最多可以有 20 个处于活动状态的作业。
6.  **作业的最大大小：**作业的大小由使用的硬盘驱动器的容量以及可在一个存储帐户中存储的最大数据量确定。每个作业可以包含不超过 10 个硬盘驱动器。

## 在管理门户中创建导入作业

创建导入作业以便通知导入/导出服务你要将一个或多个包含数据的驱动器运送到数据中心以便导入到你的存储帐户中。

### 准备驱动器

在创建导入作业前，使用 Microsoft Azure 导入/导出工具准备你的驱动器。有关使用 Microsoft Azure 导入/导出工具的更多详细信息，请参阅 [Microsoft Azure 导入/导出工具参考][Azure 导入/导出服务 REST API 参考]。你可以以独立软件包的方式下载 [Microsoft Azure 导入/导出工具][]。

若要准备你的驱动器，请按照以下三个步骤执行：

1.  确定要导入的数据，以及你将需要的驱动器的数量。
2.  确定 Azure Blob 服务中用于你的数据的目标 Blob。
3.  使用 Microsoft Azure 导入/导出工具将你的数据复制到一个或多个硬盘驱动器。

对于每个驱动器，在准备它时，Microsoft Azure 导入/导出工具会生成一个“驱动器日志”**文件。该驱动器日志文件存储于你的本地计算机上，而不是存储于驱动器本身。你在创建导入作业时将上载该日志文件。驱动器日志文件包含驱动器 ID 和 BitLocker 密钥，以及与驱动器有关的其他信息。

### 创建导入作业

1.  在准备好你的驱动器后，在管理门户中导航到你的存储帐户，并且查看仪表板。在**“速览”**下，单击**“创建导入作业”**。

2.  在向导的步骤 1 中，指示你已准备好了驱动器并且具有可用的驱动器日志文件。

3.  在步骤 2 中，提供负责该导入作业的人员的联系信息。如果你希望保存导入作业的详细日志数据，则选中**“将详细日志保存在我的‘waimportexport’Blob 容器中”**选项。

4.  在步骤 3 中，上载你在驱动器准备步骤中获取的驱动器日志文件。你需要为已准备好的每个驱动器上载一个文件。

    ![创建导入作业 - 步骤 3][]

5.  在步骤 4 中，为导入作业输入一个描述性名称。请注意，你输入的名称只能包含小写字母、数字、连字符和下划线，必须以字母开头并且不得包含空格。在作业进行中以及在作业完成后，你将使用所选名称来跟踪作业。

    接下来，从列表中选择你的数据中心区域。数据中心区域会指示必须将你的包裹运送到的数据中心和地址。有关详细信息，请参阅下文中的“常见问题”。

6.  在步骤 5 中，从列表中选择回程承运人，并输入你的承运人帐号。当你的导入作业完成后，Microsoft 将使用此帐户寄回你的驱动器。

    如果你有跟踪号码，则从列表中选择你的承运人，并输入你的跟踪号码。

    如果你还没有跟踪号码，请选择**“我将在发运我的包裹后提供此导入作业的发运信息”**，然后完成导入过程。

7.  若要在发送你的包裹后输入你的跟踪号码，请在管理门户中返回到你的存储帐户的**“导入/导出”**页面，从列表中选择你的作业并选择**“装运信息”**。在向导中导航并在步骤 2 中输入你的跟踪号码。

    如果作业处于“正在创建”、“正在发运”或“正在传送”状态，则你还可以在向导的第 2 步中更新你的承运人帐号。一旦作业处于“正在打包”状态，你将无法更新该作业的承运人帐号。

## 在管理门户中创建导出作业

创建导出作业以便通知导入/导出服务你要将一个或多个空驱动器运送到数据中心；这样，数据可以从你的存储帐户导出到驱动器，然后将驱动器运送给你。

1.  若要创建导出作业，请在管理门户中导航到你的存储帐户，并查看仪表板。在**“速览”**下，单击**“创建导出作业”**，并继续完成向导。

2.  在步骤 2 中，提供负责该导出作业的人员的联系信息。如果你希望保存导出作业的详细日志数据，则选中**“将详细日志保存在我的‘waimportexport’Blob 容器中”**选项。

3.  在步骤 3 中，指定要从你的存储帐户导出到空驱动器中的 Blob 数据。你可以选择导出该存储帐户中的所有 Blob 数据，也可以指定要导出的 Blob 或 Blob 组。

    ![创建导出作业 - 步骤 3][]

    -   若要指定要导出的 Blob，请使用**“等于”**选择器，并指定该 Blob 的相对路径，以容器名称开头。使用 *\$root* 指定根容器。
    -   若要指定以某一前缀开头的所有 Blob，请使用**“开头为”**选择器，并指定前缀，以正斜杠“/”开头。该前缀可以是容器名称的前缀、完整容器名称或者后跟 Blob 名称前缀的完整容器名称。

    下表显示有效 Blob 路径的示例：

	<table border="1" cellspacing="0" cellpadding="5" style="border: 1px solid #000000;">
	    <tbody>
	        <tr>
	<td><strong>选择器</strong></td>
	<td><strong>Blob 路径</strong></td>
	<td><strong>说明</strong></td>
	        </tr>
	        <tr>
	<td>开头为</td>
	            <td>/</td>
	<td>导出存储帐户中的所有 Blob</td>
	        </tr>
	        <tr>
	<td>开头为</td>
	<td>/$root/</td>
	<td>导出根容器中的所有 Blob</td>
	        </tr>
	        <tr>
	<td>开头为</td>
	<td>/book</td>
	<td>导出任何容器中以前缀 <strong>book</strong> 开头的所有 Blob</td>
	        </tr>
	        <tr>
	<td>开头为</td>
	<td>/music/</td>
	<td>导出容器 <strong>music</strong> 中的所有 Blob</td>
	        </tr>
	        <tr>
	<td>开头为</td>
	<td>/music/love</td>
	<td>导出容器 <strong>music</strong> 中以前缀 <strong>love</strong> 开头的所有 Blob</td>
	        </tr>
	        <tr>
	<td>等于</td>
	<td>$root/logo.bmp</td>
	<td>导出根容器中的 Blob <strong>logo.bmp</strong></td>
	        </tr>
	        <tr>
	<td>等于</td>
	<td>videos/story.mp4</td>
	<td>导出容器 <strong>videos</strong> 中的 Blob <strong>story.mp4</strong></td>
	        </tr>
	    </tbody>
	</table>

4.  在步骤 4 中，为导出作业输入一个描述性名称。你输入的名称只能包含小写字母、数字、连字符和下划线，必须以字母开头并且不得包含空格。

    数据中心区域将指示必须将你的包装运送到的数据中心。有关详细信息，请参阅下文中的“常见问题”。

5.  在步骤 5 中，从列表中选择回程承运人，并输入你的承运人帐号。当你的导出作业完成后，Microsoft 将使用此帐户寄回你的驱动器。

    如果你有跟踪号码，则从列表中选择你的承运人，并输入你的跟踪号码。

    如果你还没有跟踪号码，请选择**“我将在发运我的包裹后提供此导出作业的发运信息”**，然后完成导出过程。

6.  若要在发送你的包裹后输入你的跟踪号码，请在管理门户中返回到你的存储帐户的**“导入/导出”**页面，从列表中选择你的作业并选择**“装运信息”**。在向导中导航并在步骤 2 中输入你的跟踪号码。

    如果作业处于“正在创建”、“正在发运”或“正在传送”状态，则你还可以在向导的第 2 步中更新你的承运人帐号。一旦作业处于“正在打包”状态，你将无法更新该作业的承运人帐号。

## 管理门户中的跟踪作业状态

你可以从管理门户跟踪你的导入或导出作业的状态。导航到管理门户中的存储帐户，然后单击“导入/导出” 选项卡。你的作业的列表将出现在该页上。你可以根据作业状态、作业名称、作业类型或跟踪号码筛选该列表。

下表描述每个作业状态所代表的含义：

<table border="1" cellspacing="0" cellpadding="5" style="border: 1px solid #000000;">
    <tbody>
        <tr>
<td><strong>作业状态</strong></td>
<td><strong>说明</strong></td>
        </tr>
        <tr>
<td>创建</td>
<td>你的作业已创建，但尚未提供装运信息。</td>
        </tr>
        <tr>
<td>装运</td>
<td>你的作业已创建，并且提供了装运信息。</td>
        </tr>
        <tr>
<td>转移</td>
<td>你的数据正在从你的硬盘驱动器传输（对于导入作业）或传输到你的硬盘驱动器（对于导出作业）。</td>
        </tr>
        <tr>
<td>打包</td>
<td>你的数据传输已完成，并且你的硬盘驱动器正准备返还给你。</td>
        </tr>
        <tr>
<td>完成</td>
<td>你的硬盘驱动器已返还给你。</td>
        </tr>
    </tbody>
</table>

## 查看导出作业的 BitLocker 密钥

对于导出作业，你可以查看和复制该服务为你的驱动器生成的 BitLocker 密钥，以便你可以在从 Azure 数据中心接收驱动器后对你的导出数据进行解密。导航到管理门户中的存储帐户，然后单击“导入/导出” 选项卡。从列表中选择你的导出作业，然后单击“查看密钥” 按钮。BitLocker 密钥随即出现，如下所示：

![查看导出作业的 BitLocker 密钥][]

## 常见问题

### 常规

**针对导入/导出服务的定价是什么？**

-   有关定价信息，请参阅[定价页][]。

**导入或导出我的数据将会用多长时间？**

-   该时间将是装运磁盘所用的时间，外加要复制的每 TB 的数据几小时。

**支持哪些接口类型？**

-   导入/导出服务支持 3.5 英寸 SATA II/III 内部硬盘驱动器磁盘 (HDD)。你可以在装运前使用以下转换器将 USB 中的设备数据传输到 SATA：

    -   Anker 68UPSATAA-02BU
    -   Anker 68UPSHHDS-BU
    -   Startech SATADOCK22UE

> [WACOM.NOTE] 如果你有上方没有列出的转换器，则在购买受支持的转换器之前，可以尝试使用你的转换器运行 Microsoft Azure 导入/导出工具来准备驱动器并看看它是否工作。

**如果我想要导入或导出超过 10 个驱动器，我应该做什么？**

-   对于导入/导出服务，一个导入或导出作业在单个作业中只能引用 10 个驱动器。如果你想要装运超过 10 个驱动器，可以创建多个作业。

**如果我无意中发送了不符合支持的要求的 HDD，会发生什么情况？**

-   Azure 数据中心会将不符合支持要求的驱动器返还给你。如果包裹中只有某些驱动器满足支持要求，将处理这些驱动器，不符合支持要求的驱动器将返还给你。

### 导入/导出作业管理

**如果删除我的 Azure 存储帐户，对于我的导入和导出作业，会发生什么情况？**

-   在删除你的存储帐户时，所有 Azure 导入/导出作业将随同你的帐户一起删除。

**是否可以取消我的作业？**

-   你可以取消状态为“创建”或“装运”的作业。

**在管理门户中我可以查看多长时间的已完成作业的状态？**

-   你可以查看最长 90 天的已完成作业的状态。所有已完成作业都将在 90 天后被删除。

**Bitlocker 加密是否为必须遵守的要求？**

-   是的。所有驱动器都必须使用 BitLocker 密钥进行加密。

**是否要在返还驱动器前将驱动器格式化？**

-   不需要。所有驱动器都必须是准备了 BitLocker 的。

### 装运

**支持哪些快递服务？**

-   对于美国和欧洲的区域，仅支持 [Federal Express][] (FedEx)。所有包裹都将通过 FedEx Ground 或 FedEx International Economy 返还。

-   对于亚洲的区域，仅支持 [DHL][]。所有包裹都将通过 DHL Express Worldwide 返还。

    **重要说明**

    必须向 Azure 导入/导出服务提供你的跟踪号码；否则，将无法处理你的作业。

**是否存在与退还装运相关联的任何成本？**

-   Microsoft 在创建作业以将驱动器从数据中心发运到你的返回地址时，将使用你提供的承运人帐号。请确保提供数据中心区域内的受支持承运人的承运人帐号。如果你没有承运人帐号，可以创建一个 [FedEx][Federal Express]（对于美国和欧洲区域）或 [DHL][]（对于亚洲区域）承运人帐户。

-   返还物品的运费将记到你的承运人帐户，具体取决于承运人。

**我从哪里可以装运我的数据以及可以将数据发送到哪里？**

-   在下列区域，导入/导出服务支持将数据导入到存储帐户以及从中导出数据：

    -   美国东部
    -   美国西部
    -   美国中北部
    -   美国中南部
    -   欧洲北部
    -   欧洲西部
    -   亚洲东部
    -   亚洲东南部
-   将向你提供你的存储帐户所在区域内的一个邮寄地址。例如，如果你居住在美国，但你的存储帐户在欧洲西部数据中心，则将向你提供欧洲的一个邮寄地址来运送驱动器。

    **重要说明**

    请注意，你发运的物理介质可能需要穿越国界。你应当负责确保你的物理介质和数据是遵照适用的法律导入和/或导出的。在发运物理介质之前，请咨询你的顾问以验证你的介质和数据是否可以合法地发运到所确定的数据中心。这将有助于确保它可以及时到达 Microsoft。

-   在发运你的包裹时，必须遵守 [Microsoft Azure 服务条款][]和 [Microsoft Azure 预览功能补充使用条款][]上的条款。

**我是否可为导入/导出作业从 Microsoft 购买驱动器？**

-   不可以。对于导入和导出作业，你将需要装运你自己的驱动器。

**我应当在包裹中包括哪些内容？**

-   请仅发运你的硬盘驱动器。不要包括电源线或 USB 电缆之类的物品。

  [Azure 导入/导出服务 REST API 参考]: http://msdn.microsoft.com/zh-cn/library/dn529096.aspx
  [如何创建存储帐户]: ../storage-create-storage-account/
  [Microsoft Azure 导入/导出工具]: http://go.microsoft.com/fwlink/?LinkID=301900&clcid=0x409
  [创建导入作业 - 步骤 3]: ./media/storage-import-export-service/import-job-03.png
  [创建导出作业 - 步骤 3]: ./media/storage-import-export-service/export-job-03.png
  [查看导出作业的 BitLocker 密钥]: ./media/storage-import-export-service/export-job-bitlocker-keys.png
  [定价页]: http://www.windowsazure.cn/zh-cn/pricing/overview/
  [Federal Express]: http://www.fedex.com/us/oadr/
  [DHL]: http://www.dhl-welcome.com/Tutorial/
  [Microsoft Azure 服务条款]: http://www.windowsazure.cn/zh-cn/support/legal/services-terms/
  [Microsoft Azure 预览功能补充使用条款]: http://azure.microsoft.com/en-us/support/legal/preview-supplemental-terms/
