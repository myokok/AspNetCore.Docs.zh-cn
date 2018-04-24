---
title: "ASP.NET Core MVC Web API 中的自定义格式化程序"
author: tdykstra
description: "了解如何为 ASP.NET Core 中的 Web API 创建和使用自定义格式化程序。"
manager: wpickett
ms.author: tdykstra
ms.date: 02/08/2017
ms.prod: asp.net-core
ms.technology: aspnet
ms.topic: article
uid: mvc/models/custom-formatters
ms.openlocfilehash: 8a42f2d885bd0a0c6d2bd05f9c589def2e15d50a
ms.sourcegitcommit: a510f38930abc84c4b302029d019a34dfe76823b
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/30/2018
---
# <a name="custom-formatters-in-aspnet-core-mvc-web-apis"></a><span data-ttu-id="751f8-103">ASP.NET Core MVC Web API 中的自定义格式化程序</span><span class="sxs-lookup"><span data-stu-id="751f8-103">Custom formatters in ASP.NET Core MVC web APIs</span></span>

<span data-ttu-id="751f8-104">作者：[Tom Dykstra](https://github.com/tdykstra)</span><span class="sxs-lookup"><span data-stu-id="751f8-104">By [Tom Dykstra](https://github.com/tdykstra)</span></span>

<span data-ttu-id="751f8-105">ASP.NET Core MVC 使用 JSON、XML 或纯文本格式，为 Web API 中的数据交换提供内置支持。</span><span class="sxs-lookup"><span data-stu-id="751f8-105">ASP.NET Core MVC has built-in support for data exchange in web APIs by using JSON, XML, or plain text formats.</span></span> <span data-ttu-id="751f8-106">本文展示如何通过创建自定义格式化程序，添加对其他格式的支持。</span><span class="sxs-lookup"><span data-stu-id="751f8-106">This article shows how to add support for additional formats by creating custom formatters.</span></span>

<span data-ttu-id="751f8-107">[查看或下载 GitHub 中的示例](https://github.com/aspnet/Docs/tree/master/aspnetcore/mvc/advanced/custom-formatters/sample)。</span><span class="sxs-lookup"><span data-stu-id="751f8-107">[View or download sample from GitHub](https://github.com/aspnet/Docs/tree/master/aspnetcore/mvc/advanced/custom-formatters/sample).</span></span>

## <a name="when-to-use-custom-formatters"></a><span data-ttu-id="751f8-108">何时使用自定义格式化程序</span><span class="sxs-lookup"><span data-stu-id="751f8-108">When to use custom formatters</span></span>

<span data-ttu-id="751f8-109">如果希望[内容协商](xref:mvc/models/formatting)过程支持内置格式化程序（JSON、XML 和纯文本）所不支持的内容类型，可使用自定义格式化程序。</span><span class="sxs-lookup"><span data-stu-id="751f8-109">Use a custom formatter when you want the [content negotiation](xref:mvc/models/formatting) process to support a content type that isn't supported by the built-in formatters (JSON, XML, and plain text).</span></span>

<span data-ttu-id="751f8-110">例如，如果 Web API 的某些客户端可以处理 [Protobuf](https://github.com/google/protobuf) 格式，你可能想在这些客户端上使用 Protobuf，因为它更高效。</span><span class="sxs-lookup"><span data-stu-id="751f8-110">For example, if some of the clients for your web API can handle the [Protobuf](https://github.com/google/protobuf) format, you might want to use Protobuf with those clients because it's more efficient.</span></span>  <span data-ttu-id="751f8-111">或者，你可能希望 Web API 使用 [vCard](https://wikipedia.org/wiki/VCard) 格式发送联系人姓名和地址，这种格式经常用于交换联系人数据。</span><span class="sxs-lookup"><span data-stu-id="751f8-111">Or you might want your web API to send contact names and addresses in [vCard](https://wikipedia.org/wiki/VCard) format, a commonly used format for exchanging contact data.</span></span> <span data-ttu-id="751f8-112">本文提供的示例应用可实现简单的 vCard 格式化程序。</span><span class="sxs-lookup"><span data-stu-id="751f8-112">The sample app provided with this article implements a simple vCard formatter.</span></span>

## <a name="overview-of-how-to-use-a-custom-formatter"></a><span data-ttu-id="751f8-113">有关如何使用自定义格式化程序的概述</span><span class="sxs-lookup"><span data-stu-id="751f8-113">Overview of how to use a custom formatter</span></span>

<span data-ttu-id="751f8-114">创建和使用自定义格式化程序的步骤如下：</span><span class="sxs-lookup"><span data-stu-id="751f8-114">Here are the steps to create and use a custom formatter:</span></span>

* <span data-ttu-id="751f8-115">如果想对要发送到客户端的数据进行序列化，则创建输出格式化程序类。</span><span class="sxs-lookup"><span data-stu-id="751f8-115">Create an output formatter class if you want to serialize data to send to the client.</span></span>
* <span data-ttu-id="751f8-116">如果想对从客户端接收的数据进行反序列化，则创建输入格式化程序类。</span><span class="sxs-lookup"><span data-stu-id="751f8-116">Create an input formatter class if you want to deserialize data received from the client.</span></span> 
* <span data-ttu-id="751f8-117">将格式化程序的实例添加到 [MvcOptions](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.mvc.mvcoptions) 中的 `InputFormatters` 和 `OutputFormatters` 集合。</span><span class="sxs-lookup"><span data-stu-id="751f8-117">Add instances of your formatters to the `InputFormatters` and `OutputFormatters` collections in [MvcOptions](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.mvc.mvcoptions).</span></span>

<span data-ttu-id="751f8-118">以下部分针对其中每个步骤提供了指南和代码示例。</span><span class="sxs-lookup"><span data-stu-id="751f8-118">The following sections provide guidance and code examples for each of these steps.</span></span>

## <a name="how-to-create-a-custom-formatter-class"></a><span data-ttu-id="751f8-119">如何创建自定义格式化程序类</span><span class="sxs-lookup"><span data-stu-id="751f8-119">How to create a custom formatter class</span></span>

<span data-ttu-id="751f8-120">若要创建格式化程序，请执行以下操作：</span><span class="sxs-lookup"><span data-stu-id="751f8-120">To create a formatter:</span></span>

* <span data-ttu-id="751f8-121">从相应的基类中派生类。</span><span class="sxs-lookup"><span data-stu-id="751f8-121">Derive the class from the appropriate base class.</span></span>
* <span data-ttu-id="751f8-122">在构造函数中指定有效的媒体类型和编码。</span><span class="sxs-lookup"><span data-stu-id="751f8-122">Specify valid media types and encodings in the constructor.</span></span>
* <span data-ttu-id="751f8-123">重写 `CanReadType`/`CanWriteType` 方法</span><span class="sxs-lookup"><span data-stu-id="751f8-123">Override `CanReadType`/`CanWriteType` methods</span></span>
* <span data-ttu-id="751f8-124">重写 `ReadRequestBodyAsync`/`WriteResponseBodyAsync` 方法</span><span class="sxs-lookup"><span data-stu-id="751f8-124">Override `ReadRequestBodyAsync`/`WriteResponseBodyAsync` methods</span></span>
  
### <a name="derive-from-the-appropriate-base-class"></a><span data-ttu-id="751f8-125">从相应的基类中派生</span><span class="sxs-lookup"><span data-stu-id="751f8-125">Derive from the appropriate base class</span></span>

<span data-ttu-id="751f8-126">对于文本媒体类型（例如，vCard），从 [TextInputFormatter](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.mvc.formatters.textinputformatter) 或 [TextOutputFormatter](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.mvc.formatters.textoutputformatter) 基类派生。</span><span class="sxs-lookup"><span data-stu-id="751f8-126">For text media types (for example, vCard), derive from the [TextInputFormatter](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.mvc.formatters.textinputformatter) or [TextOutputFormatter](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.mvc.formatters.textoutputformatter) base class.</span></span>

[!code-csharp[Main](custom-formatters/sample/Formatters/VcardOutputFormatter.cs?name=classdef)]

<span data-ttu-id="751f8-127">对于二进制类型，从 [InputFormatter](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.mvc.formatters.inputformatter) 或 [OutputFormatter](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.mvc.formatters.outputformatter) 基类派生。</span><span class="sxs-lookup"><span data-stu-id="751f8-127">For binary types, derive from the [InputFormatter](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.mvc.formatters.inputformatter) or [OutputFormatter](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.mvc.formatters.outputformatter) base class.</span></span>

### <a name="specify-valid-media-types-and-encodings"></a><span data-ttu-id="751f8-128">指定有效的媒体类型和编码</span><span class="sxs-lookup"><span data-stu-id="751f8-128">Specify valid media types and encodings</span></span>

<span data-ttu-id="751f8-129">在构造函数中，通过添加到 `SupportedMediaTypes` 和 `SupportedEncodings` 集合来指定有效的媒体类型和编码。</span><span class="sxs-lookup"><span data-stu-id="751f8-129">In the constructor, specify valid media types and encodings by adding to the `SupportedMediaTypes` and `SupportedEncodings` collections.</span></span>

[!code-csharp[Main](custom-formatters/sample/Formatters/VcardOutputFormatter.cs?name=ctor&highlight=3,5-6)]

> [!NOTE]  
> <span data-ttu-id="751f8-130">不能在格式化程序类中执行构造函数依赖关系注入。</span><span class="sxs-lookup"><span data-stu-id="751f8-130">You can't do constructor dependency injection in a formatter class.</span></span> <span data-ttu-id="751f8-131">例如，不能通过向构造函数添加记录器参数来获取记录器。</span><span class="sxs-lookup"><span data-stu-id="751f8-131">For example, you can't get a logger by adding a logger parameter to the constructor.</span></span> <span data-ttu-id="751f8-132">若要访问服务，必须使用传递到方法的上下文对象。</span><span class="sxs-lookup"><span data-stu-id="751f8-132">To access services, you have to use the context object that gets passed in to your methods.</span></span> <span data-ttu-id="751f8-133">[下面](#read-write)的代码示例展示了如何执行此操作。</span><span class="sxs-lookup"><span data-stu-id="751f8-133">A code example [below](#read-write) shows how to do this.</span></span>

### <a name="override-canreadtypecanwritetype"></a><span data-ttu-id="751f8-134">重写 CanReadType/CanWriteType</span><span class="sxs-lookup"><span data-stu-id="751f8-134">Override CanReadType/CanWriteType</span></span> 

<span data-ttu-id="751f8-135">通过重写 `CanReadType` 或 `CanWriteType` 方法，指定可反序列化为或从其序列化的类型。</span><span class="sxs-lookup"><span data-stu-id="751f8-135">Specify the type you can deserialize into or serialize from by overriding the `CanReadType` or `CanWriteType` methods.</span></span> <span data-ttu-id="751f8-136">例如，可能只能从 `Contact` 类型创建 vCard 文本，反之亦然。</span><span class="sxs-lookup"><span data-stu-id="751f8-136">For example, you might only be able to create vCard text from a `Contact` type and vice versa.</span></span>

[!code-csharp[Main](custom-formatters/sample/Formatters/VcardOutputFormatter.cs?name=canwritetype)]

#### <a name="the-canwriteresult-method"></a><span data-ttu-id="751f8-137">CanWriteResult 方法</span><span class="sxs-lookup"><span data-stu-id="751f8-137">The CanWriteResult method</span></span>

<span data-ttu-id="751f8-138">在某些情况下，必须重写 `CanWriteResult`，而不是 `CanWriteType`。</span><span class="sxs-lookup"><span data-stu-id="751f8-138">In some scenarios you have to override `CanWriteResult` instead of `CanWriteType`.</span></span> <span data-ttu-id="751f8-139">如果满足以下条件，则使用 `CanWriteResult`：</span><span class="sxs-lookup"><span data-stu-id="751f8-139">Use `CanWriteResult` if the following conditions are true:</span></span>

  * <span data-ttu-id="751f8-140">操作方法返回模型类。</span><span class="sxs-lookup"><span data-stu-id="751f8-140">Your action method returns a model class.</span></span>
  * <span data-ttu-id="751f8-141">具有可能在运行时返回的派生类。</span><span class="sxs-lookup"><span data-stu-id="751f8-141">There are derived classes which might be returned at runtime.</span></span>
  * <span data-ttu-id="751f8-142">需要知道操作在运行时返回了哪个派生类。</span><span class="sxs-lookup"><span data-stu-id="751f8-142">You need to know at runtime which derived class was returned by the action.</span></span>  

<span data-ttu-id="751f8-143">例如，假设操作方法签名返回 `Person` 类型，但它可能返回从 `Person` 派生的 `Student` 或 `Instructor` 类型。</span><span class="sxs-lookup"><span data-stu-id="751f8-143">For example, suppose your action method signature returns a `Person` type, but it may return a `Student` or `Instructor` type that derives from `Person`.</span></span> <span data-ttu-id="751f8-144">如果希望格式化程序仅处理 `Student` 对象，请检查提供给 `CanWriteResult` 方法的上下文对象中的[对象](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.mvc.formatters.outputformattercanwritecontext#Microsoft_AspNetCore_Mvc_Formatters_OutputFormatterCanWriteContext_Object)类型。</span><span class="sxs-lookup"><span data-stu-id="751f8-144">If you want your formatter to handle only `Student` objects, check the type of [Object](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.mvc.formatters.outputformattercanwritecontext#Microsoft_AspNetCore_Mvc_Formatters_OutputFormatterCanWriteContext_Object) in the context object provided to the `CanWriteResult` method.</span></span> <span data-ttu-id="751f8-145">请注意，当操作方法返回 `IActionResult` 时，不必使用 `CanWriteResult`；在这种情况下，`CanWriteType` 方法可接收运行时类型。</span><span class="sxs-lookup"><span data-stu-id="751f8-145">Note that it's not necessary to use `CanWriteResult` when the action method returns `IActionResult`; in that case, the `CanWriteType` method receives the runtime type.</span></span>

<a id="read-write"></a>
### <a name="override-readrequestbodyasyncwriteresponsebodyasync"></a><span data-ttu-id="751f8-146">重写 ReadRequestBodyAsync/WriteResponseBodyAsync</span><span class="sxs-lookup"><span data-stu-id="751f8-146">Override ReadRequestBodyAsync/WriteResponseBodyAsync</span></span> 

<span data-ttu-id="751f8-147">实际的反序列化或序列化工作在 `ReadRequestBodyAsync` 或 `WriteResponseBodyAsync` 中执行。</span><span class="sxs-lookup"><span data-stu-id="751f8-147">You do the actual work of deserializing or serializing in `ReadRequestBodyAsync` or `WriteResponseBodyAsync`.</span></span>  <span data-ttu-id="751f8-148">以下示例中突出显示的行展示了如何从依赖关系注入容器中获取服务（不能从构造函数参数中获取它们）。</span><span class="sxs-lookup"><span data-stu-id="751f8-148">The highlighted lines in the following example show how to get services from the dependency injection container (you can't get them from constructor parameters).</span></span>

[!code-csharp[Main](custom-formatters/sample/Formatters/VcardOutputFormatter.cs?name=writeresponse&highlight=3-4)]

## <a name="how-to-configure-mvc-to-use-a-custom-formatter"></a><span data-ttu-id="751f8-149">如何将 MVC 配置为使用自定义格式化程序</span><span class="sxs-lookup"><span data-stu-id="751f8-149">How to configure MVC to use a custom formatter</span></span>
 
<span data-ttu-id="751f8-150">若要使用自定义格式化程序，请将格式化程序类的实例添加到 `InputFormatters` 或 `OutputFormatters` 集合。</span><span class="sxs-lookup"><span data-stu-id="751f8-150">To use a custom formatter, add an instance of the formatter class to the `InputFormatters` or `OutputFormatters` collection.</span></span>

[!code-csharp[Main](custom-formatters/sample/Startup.cs?name=mvcoptions&highlight=3-4)]

<span data-ttu-id="751f8-151">按格式化程序的插入顺序对其进行计算。</span><span class="sxs-lookup"><span data-stu-id="751f8-151">Formatters are evaluated in the order you insert them.</span></span> <span data-ttu-id="751f8-152">第一个优先。</span><span class="sxs-lookup"><span data-stu-id="751f8-152">The first one takes precedence.</span></span> 

## <a name="next-steps"></a><span data-ttu-id="751f8-153">后续步骤</span><span class="sxs-lookup"><span data-stu-id="751f8-153">Next steps</span></span>

<span data-ttu-id="751f8-154">请参阅[示例应用程序](https://github.com/aspnet/Docs/tree/master/aspnetcore/mvc/advanced/custom-formatters/sample)，它可实现简单的 vCard 输入和输出格式化程序。</span><span class="sxs-lookup"><span data-stu-id="751f8-154">See the [sample application](https://github.com/aspnet/Docs/tree/master/aspnetcore/mvc/advanced/custom-formatters/sample), which implements simple vCard input and output formatters.</span></span>  <span data-ttu-id="751f8-155">该应用程序可读取和写入与以下示例类似的 vCard：</span><span class="sxs-lookup"><span data-stu-id="751f8-155">The application reads and writes vCards that look like the following example:</span></span>

```
BEGIN:VCARD
VERSION:2.1
N:Davolio;Nancy
FN:Nancy Davolio
UID:20293482-9240-4d68-b475-325df4a83728
END:VCARD
```

<span data-ttu-id="751f8-156">若要查看 vCard 输出，请运行该应用程序，并向 `http://localhost:63313/api/contacts/`（从 Visual Studio 运行时）或 `http://localhost:5000/api/contacts/`（从命令行运行时）发送具有 Accept 标头“text/vcard”的 Get 请求。</span><span class="sxs-lookup"><span data-stu-id="751f8-156">To see vCard output, run the application and send a Get request with Accept header "text/vcard" to `http://localhost:63313/api/contacts/` (when running from Visual Studio) or `http://localhost:5000/api/contacts/` (when running from the command line).</span></span>

<span data-ttu-id="751f8-157">若要将 vCard 添加到内存中联系人集合，请向相同的 URL 发送具有 Content-Type 标头“text/vcard”且正文中包含 vCard 文本的 Post 请求，格式化方式与上面的示例类似。</span><span class="sxs-lookup"><span data-stu-id="751f8-157">To add a vCard to the in-memory collection of contacts, send a Post request to the same URL, with Content-Type header "text/vcard" and with vCard text in the body, formatted like the example above.</span></span>