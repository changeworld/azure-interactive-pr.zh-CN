Azure Blob 存储是一种低成本并可大规模缩放的服务，可以用来托管静态文件。 在本教程中，可以使用它为生成的 Web 应用提供静态内容（例如 HTML、JavaScript、CSS）。

## <a name="create-a-storage-account"></a>创建存储帐户

存储帐户是 Azure 资源，用于存储表、队列、文件、Blob（对象）和虚拟机磁盘。

1. 选择“进入焦点模式”按钮，登录到 Cloud Shell (Bash)。 此按钮位于页面右上角或底部，具体取决于浏览器窗口的宽度。 焦点模式将 Cloud Shell 窗口停靠在浏览器窗口右侧，便于执行教程中显示的命令。

1. 在 Azure 中，资源组是一个容器，用于保存相关的 Azure 资源，目的是方便管理。 创建名为 **first-serverless-app** 的新资源组。

    ```azurecli
    az group create -n first-serverless-app -l westcentralus
    ```

1. 本教程的静态内容（HTML、CSS 和 JavaScript 文件）托管在 Blob 存储中。 Blob 存储需要存储帐户。 在资源组中创建存储帐户（常规用途 V2）。 将 `<storage account name>` 替换为唯一名称。

    ```azurecli
    az storage account create -n <storage account name> -g first-serverless-app --kind StorageV2 -l westcentralus --https-only true --sku Standard_LRS
    ```

1. 使用 [Azure 门户](https://portal.azure.com)顶部的搜索栏找到刚创建的存储帐户并将其打开。

1. 在左导航栏中选择“静态网站(预览版)”，配置用于静态网站托管的容器。
    - 选择“启用”以启用静态网站。
    - 输入 *index.html* 作为索引文档名称。 此字段已经有灰色字体的 *index.html*，但这仅仅是示例文本；你仍然需要在字段中输入该值。
    - 单击“ **保存**”。
    
    ![输入静态网站设置](media/functions-first-serverless-web-app/1-storage-static-website.png)

1. 将“主终结点”保存在某个位置，确保在整个教程中都可以方便地从其进行复制。 这是 Web 应用程序的 URL。

## <a name="upload-the-web-application"></a>上传 Web 应用程序

1. 在本教程中生成的应用程序的源文件位于 [GitHub 存储库](https://github.com/Azure-Samples/functions-first-serverless-web-application)中。 确保处于 Cloud Shell 的主目录中，然后克隆此存储库。

    ```azurecli
    cd ~
    git clone https://github.com/Azure-Samples/functions-first-serverless-web-application
    ```

    存储库克隆到 `/home/<username>/functions-first-serverless-web-application`。

1. 客户端 Web 应用程序位于 **www** 文件夹中，是使用 Vue.js JavaScript 框架生成的。 转到该文件夹中并运行 npm 命令，以便安装应用程序的依赖项并生成应用程序。 这些命令中的最后一个可能需要几分钟才能完成。

    ```azurecli
    cd ~/functions-first-serverless-web-application/www
    npm install
    npm run generate
    ```

    应用程序在 **dist** 文件夹中生成。

1. 将当前目录更改到 **dist**，然后将应用程序上传到 **$web** Blob 容器。

    ```azurecli
    cd dist
    az storage blob upload-batch -s . -d \$web --account-name <storage account name>
    ```

1. 在 Web 浏览器中打开存储静态网站主终结点 URL，以便查看应用程序。

    ![首个无服务器 Web 应用主页](media/functions-first-serverless-web-app/1-app-screenshot-new.png)


## <a name="summary"></a>摘要

在本模块中，你创建了名为 **first-serverless-app** 的资源组，其中包含一个存储帐户。 在存储帐户中，名为 **$web** 的 Blob 容器可以为 Web 应用程序存储静态内容，并使该内容公开可用。 接下来，请学习如何使用无服务器函数，将图像从此 Web 应用程序上传到 Blob 存储。
