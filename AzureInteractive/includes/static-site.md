<span data-ttu-id="d071e-101">Azure Blob 存储是一种低成本并可大规模缩放的服务，可以用来托管静态文件。</span><span class="sxs-lookup"><span data-stu-id="d071e-101">Azure Blob storage is a low-cost and massively scalable service that can be used to host static files.</span></span> <span data-ttu-id="d071e-102">在本教程中，可以使用它为生成的 Web 应用提供静态内容（例如 HTML、JavaScript、CSS）。</span><span class="sxs-lookup"><span data-stu-id="d071e-102">For this tutorial, you use it to serve static content (for example, HTML, JavaScript, CSS) for the web app that you build.</span></span>

## <a name="create-a-storage-account"></a><span data-ttu-id="d071e-103">创建存储帐户</span><span class="sxs-lookup"><span data-stu-id="d071e-103">Create a Storage account</span></span>

<span data-ttu-id="d071e-104">存储帐户是 Azure 资源，用于存储表、队列、文件、Blob（对象）和虚拟机磁盘。</span><span class="sxs-lookup"><span data-stu-id="d071e-104">A Storage account is an Azure resource that allows you to store tables, queues, files, blobs (objects), and virtual machine disks.</span></span>

1. <span data-ttu-id="d071e-105">选择“进入焦点模式”按钮，登录到 Cloud Shell (Bash)。</span><span class="sxs-lookup"><span data-stu-id="d071e-105">Log in to the Cloud Shell (Bash), by selecting the **Enter focus mode** button.</span></span> <span data-ttu-id="d071e-106">此按钮位于页面右上角或底部，具体取决于浏览器窗口的宽度。</span><span class="sxs-lookup"><span data-stu-id="d071e-106">This button is at the top right or the bottom of the page, depending on how wide your browser window is.</span></span> <span data-ttu-id="d071e-107">焦点模式将 Cloud Shell 窗口停靠在浏览器窗口右侧，便于执行教程中显示的命令。</span><span class="sxs-lookup"><span data-stu-id="d071e-107">Focus mode docks a Cloud Shell window on the right side of your browser window, so you can easily execute commands that are shown in the tutorial.</span></span>

1. <span data-ttu-id="d071e-108">在 Azure 中，资源组是一个容器，用于保存相关的 Azure 资源，目的是方便管理。</span><span class="sxs-lookup"><span data-stu-id="d071e-108">In Azure, a Resource Group is a container that holds related Azure resources for ease of management.</span></span> <span data-ttu-id="d071e-109">创建名为 **first-serverless-app** 的新资源组。</span><span class="sxs-lookup"><span data-stu-id="d071e-109">Create a new resource group named **first-serverless-app**.</span></span>

    ```azurecli
    az group create -n first-serverless-app -l westcentralus
    ```

1. <span data-ttu-id="d071e-110">本教程的静态内容（HTML、CSS 和 JavaScript 文件）托管在 Blob 存储中。</span><span class="sxs-lookup"><span data-stu-id="d071e-110">The static content (HTML, CSS, and JavaScript files) for this tutorial is hosted in Blob Storage.</span></span> <span data-ttu-id="d071e-111">Blob 存储需要存储帐户。</span><span class="sxs-lookup"><span data-stu-id="d071e-111">Blob Storage requires a Storage account.</span></span> <span data-ttu-id="d071e-112">在资源组中创建存储帐户（常规用途 V2）。</span><span class="sxs-lookup"><span data-stu-id="d071e-112">Create a Storage account (general purpose V2) in the resource group.</span></span> <span data-ttu-id="d071e-113">将 `<storage account name>` 替换为唯一名称。</span><span class="sxs-lookup"><span data-stu-id="d071e-113">Replace `<storage account name>` with a unique name.</span></span>

    ```azurecli
    az storage account create -n <storage account name> -g first-serverless-app --kind StorageV2 -l westcentralus --https-only true --sku Standard_LRS
    ```

1. <span data-ttu-id="d071e-114">使用 [Azure 门户](https://portal.azure.com)顶部的搜索栏找到刚创建的存储帐户并将其打开。</span><span class="sxs-lookup"><span data-stu-id="d071e-114">Using the Search bar at the top of the [Azure portal](https://portal.azure.com), find the storage account you just created and open it.</span></span>

1. <span data-ttu-id="d071e-115">在左导航栏中选择“静态网站(预览版)”，配置用于静态网站托管的容器。</span><span class="sxs-lookup"><span data-stu-id="d071e-115">On the left navigation, select **Static website (preview)** to configure a container for static website hosting.</span></span>
    - <span data-ttu-id="d071e-116">选择“启用”以启用静态网站。</span><span class="sxs-lookup"><span data-stu-id="d071e-116">Select **Enabled** to enable static website.</span></span>
    - <span data-ttu-id="d071e-117">输入 *index.html* 作为索引文档名称。</span><span class="sxs-lookup"><span data-stu-id="d071e-117">Enter *index.html* as the index document name.</span></span> <span data-ttu-id="d071e-118">此字段已经有灰色字体的 *index.html*，但这仅仅是示例文本；你仍然需要在字段中输入该值。</span><span class="sxs-lookup"><span data-stu-id="d071e-118">The field already has *index.html* in a gray font but this is only example text; you still have to enter that value in the field.</span></span>
    - <span data-ttu-id="d071e-119">单击“ **保存**”。</span><span class="sxs-lookup"><span data-stu-id="d071e-119">Click **Save**.</span></span>
    
    ![输入静态网站设置](media/functions-first-serverless-web-app/1-storage-static-website.png)

1. <span data-ttu-id="d071e-121">将“主终结点”保存在某个位置，确保在整个教程中都可以方便地从其进行复制。</span><span class="sxs-lookup"><span data-stu-id="d071e-121">Save the **Primary Endpoint** somewhere you can conveniently copy it from while working through the tutorial.</span></span> <span data-ttu-id="d071e-122">这是 Web 应用程序的 URL。</span><span class="sxs-lookup"><span data-stu-id="d071e-122">This is the URL of your web application.</span></span>

## <a name="upload-the-web-application"></a><span data-ttu-id="d071e-123">上传 Web 应用程序</span><span class="sxs-lookup"><span data-stu-id="d071e-123">Upload the web application</span></span>

1. <span data-ttu-id="d071e-124">在本教程中生成的应用程序的源文件位于 [GitHub 存储库](https://github.com/Azure-Samples/functions-first-serverless-web-application)中。</span><span class="sxs-lookup"><span data-stu-id="d071e-124">The source files for the application that you build in this tutorial are located in a [GitHub repository](https://github.com/Azure-Samples/functions-first-serverless-web-application).</span></span> <span data-ttu-id="d071e-125">确保处于 Cloud Shell 的主目录中，然后克隆此存储库。</span><span class="sxs-lookup"><span data-stu-id="d071e-125">Make sure that you are in your home directory in Cloud Shell and clone this repository.</span></span>

    ```azurecli
    cd ~
    git clone https://github.com/Azure-Samples/functions-first-serverless-web-application
    ```

    <span data-ttu-id="d071e-126">存储库克隆到 `/home/<username>/functions-first-serverless-web-application`。</span><span class="sxs-lookup"><span data-stu-id="d071e-126">The repository is cloned to `/home/<username>/functions-first-serverless-web-application`.</span></span>

1. <span data-ttu-id="d071e-127">客户端 Web 应用程序位于 **www** 文件夹中，是使用 Vue.js JavaScript 框架生成的。</span><span class="sxs-lookup"><span data-stu-id="d071e-127">The client-side web application is located in the **www** folder and is built using the Vue.js JavaScript framework.</span></span> <span data-ttu-id="d071e-128">转到该文件夹中并运行 npm 命令，以便安装应用程序的依赖项并生成应用程序。</span><span class="sxs-lookup"><span data-stu-id="d071e-128">Change into the folder and run npm commands to install the application's dependencies and build the application.</span></span> <span data-ttu-id="d071e-129">这些命令中的最后一个可能需要几分钟才能完成。</span><span class="sxs-lookup"><span data-stu-id="d071e-129">The last of these commands might take several minutes to complete.</span></span>

    ```azurecli
    cd ~/functions-first-serverless-web-application/www
    npm install
    npm run generate
    ```

    <span data-ttu-id="d071e-130">应用程序在 **dist** 文件夹中生成。</span><span class="sxs-lookup"><span data-stu-id="d071e-130">The application is generated in the **dist** folder.</span></span>

1. <span data-ttu-id="d071e-131">将当前目录更改到 **dist**，然后将应用程序上传到 **$web** Blob 容器。</span><span class="sxs-lookup"><span data-stu-id="d071e-131">Change the current directory to **dist** and upload the application to the **$web** Blob container.</span></span>

    ```azurecli
    cd dist
    az storage blob upload-batch -s . -d \$web --account-name <storage account name>
    ```

1. <span data-ttu-id="d071e-132">在 Web 浏览器中打开存储静态网站主终结点 URL，以便查看应用程序。</span><span class="sxs-lookup"><span data-stu-id="d071e-132">Open the Storage static websites primary endpoint URL in a web browser to view the application.</span></span>

    ![首个无服务器 Web 应用主页](media/functions-first-serverless-web-app/1-app-screenshot-new.png)


## <a name="summary"></a><span data-ttu-id="d071e-134">摘要</span><span class="sxs-lookup"><span data-stu-id="d071e-134">Summary</span></span>

<span data-ttu-id="d071e-135">在本模块中，你创建了名为 **first-serverless-app** 的资源组，其中包含一个存储帐户。</span><span class="sxs-lookup"><span data-stu-id="d071e-135">In this module, you created a resource group named **first-serverless-app** containing a Storage account.</span></span> <span data-ttu-id="d071e-136">在存储帐户中，名为 **$web** 的 Blob 容器可以为 Web 应用程序存储静态内容，并使该内容公开可用。</span><span class="sxs-lookup"><span data-stu-id="d071e-136">A blob container named **$web** in the Storage account stores the static content for your web application and makes the content available publicly.</span></span> <span data-ttu-id="d071e-137">接下来，请学习如何使用无服务器函数，将图像从此 Web 应用程序上传到 Blob 存储。</span><span class="sxs-lookup"><span data-stu-id="d071e-137">Next, you learn how to use a serverless function to upload images to Blob storage from this web application.</span></span>
