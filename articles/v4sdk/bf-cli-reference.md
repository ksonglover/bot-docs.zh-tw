---
title: Azure Bot Framework 命令列介面 (CLI) 參考 | Microsoft Docs
description: 了解 Bot Framework 命令列介面 (CLI)。
keywords: Bot Framework 命令列介面, Bot Framework CLI
author: JonathanFingold
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 10/25/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: b516f20e74ac364a1ec0e953983701d1ab4b1e33
ms.sourcegitcommit: 4751c7b8ff1d3603d4596e4fa99e0071036c207c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 11/02/2019
ms.locfileid: "73443146"
---
# <a name="bot-framework-cli-reference"></a>Bot Framework CLI 參考

[!INCLUDE[applies-to](../includes/applies-to.md)]

使用量

```sh-session
npm install -g @microsoft/botframework-cli
```

## <a name="commands"></a>命令

* [`bf chatdown`](#bf-chatdown)
* [`bf chatdown:convert`](#bf-chatdownconvert)
* [`bf config`](#bf-config)
* [`bf config:set:qnamaker`](#bf-configsetqnamaker)
* [`bf config:set:telemetry`](#bf-configsettelemetry)
* [`bf config:show`](#bf-configshow)
* [`bf config:show:qnamaker`](#bf-configshowqnamaker)
* [`bf config:show:telemetry`](#bf-configshowtelemetry)
* [`bf help [COMMAND]`](#bf-help-command)
* [`bf luis`](#bf-luis)
* [`bf luis:convert`](#bf-luisconvert)
* [`bf luis:generate:cs`](#bf-luisgeneratecs)
* [`bf luis:generate:ts`](#bf-luisgeneratets)
* [`bf luis:translate`](#bf-luistranslate)
* [`bf qnamaker`](#bf-qnamaker)
* [`bf qnamaker:alterations`](#bf-qnamakeralterations)
* [`bf qnamaker:alterations:list`](#bf-qnamakeralterationslist)
* [`bf qnamaker:alterations:replace`](#bf-qnamakeralterationsreplace)
* [`bf qnamaker:convert`](#bf-qnamakerconvert)
* [`bf qnamaker:endpointkeys`](#bf-qnamakerendpointkeys)
* [`bf qnamaker:endpointkeys:list`](#bf-qnamakerendpointkeyslist)
* [`bf qnamaker:endpointkeys:refresh`](#bf-qnamakerendpointkeysrefresh)
* [`bf qnamaker:endpointsettings`](#bf-qnamakerendpointsettings)
* [`bf qnamaker:endpointsettings:get`](#bf-qnamakerendpointsettingsget)
* [`bf qnamaker:endpointsettings:update`](#bf-qnamakerendpointsettingsupdate)
* [`bf qnamaker:init`](#bf-qnamakerinit)
* [`bf qnamaker:kb`](#bf-qnamakerkb)
* [`bf qnamaker:kb:create`](#bf-qnamakerkbcreate)
* [`bf qnamaker:kb:delete`](#bf-qnamakerkbdelete)
* [`bf qnamaker:kb:export`](#bf-qnamakerkbexport)
* [`bf qnamaker:kb:get`](#bf-qnamakerkbget)
* [`bf qnamaker:kb:list`](#bf-qnamakerkblist)
* [`bf qnamaker:kb:publish`](#bf-qnamakerkbpublish)
* [`bf qnamaker:kb:replace`](#bf-qnamakerkbreplace)
* [`bf qnamaker:kb:update`](#bf-qnamakerkbupdate)
* [`bf qnamaker:operationdetails`](#bf-qnamakeroperationdetails)
* [`bf qnamaker:operationdetails:get`](#bf-qnamakeroperationdetailsget)
* [`bf qnamaker:query`](#bf-qnamakerquery)
* [`bf qnamaker:train`](#bf-qnamakertrain)
* [`bf qnamaker:translate`](#bf-qnamakertranslate)

## `bf chatdown`

將 \<filename>.chat 格式的聊天對話方塊檔案轉換成文字記錄檔案。 為每個 .chat 檔案寫入對應的 \<filename>.transcript。

```text
USAGE
  $ bf chatdown

OPTIONS
  -h, --help  Chatdown command help
```

_參閱程式碼：[@microsoft/bf-chatdown](https://github.com/microsoft/botframework-cli/tree/master/packages/chatdown/src/commands/chatdown/index.ts)_

## `bf chatdown:convert`

將 \<filename>.chat 格式的聊天對話方塊檔案轉換成文字記錄檔案。 為每個 .chat 檔案寫入對應的 \<filename>.transcript。

```text
USAGE
  $ bf chatdown:convert

OPTIONS
  -f, --force    If --out flag is provided with the path to an existing file, overwrites that file
  -h, --help     Chatdown command help

  -i, --in=in    The path of the chat file or directory to be parsed. A glob expression may be passed containing chat
                 files to be processed all at once, ex. ./**/*.chat. If flag is omitted, stdin will be used. If an
                 output directory is not present (-o), it will default the output to the current working directory.

  -o, --out=out  Path to the directory where the output of the multiple chat file processing (-o) will be placed.

  -p, --prefix   Prefix stdout with package name.

  -s, --stamp    Use static timestamps when generating timestamps on activities.

EXAMPLE

     $ bf chatdown
     $ bf chatdown --in=./path/to/file/sample.chat
     $ bf chatdown --in ./test/utils/*.sample.chat -o ./
     $ (echo user=Joe && [ConversationUpdate=MembersAdded=Joe]) | bf chatdown --static
```

_參閱程式碼：[@microsoft/bf-chatdown](https://github.com/microsoft/botframework-cli/tree/master/packages/chatdown/src/commands/chatdown/convert.ts)_

## `bf config`

設定 CLI 內的各種設定。

```text
USAGE
  $ bf config

OPTIONS
  -h, --help  config help
```

_參閱程式碼：[@microsoft/bf-cli-config](https://github.com/microsoft/botframework-cli/tree/master/packages/config/src/commands/config/index.ts)_

## `bf config:set:qnamaker`

設定 QnA Maker 的 config 資料

```text
USAGE
  $ bf config:set:qnamaker

OPTIONS
  -h, --help                         config:set:qnamaker help
  --endpointKey=endpointKey          QnAMaker endpointKey to be set
  --hostname=hostname                QnAMaker hostname to be set
  --kbId=kbId                        QnAMaker kbId to be set
  --subscriptionKey=subscriptionKey  QnAMaker subscriptionkey to be set

EXAMPLE

     {
       "qnamaker_kbId": "3bda64af-dddd-dddd-dddd-021906b093b1",
       "qnamaker_subscriptionKey": "nnnnnnnnnnnnnnnnnnnnnnnnn",
       "qnamaker_endpointKey": "6b5ecf9c-kkkk-kkkk-kkkk-761489817e5f",
       "qnamaker_hostname": "https://{qnaservice-hostname}.azurewebsites.net"
     }
```

_參閱程式碼：[@microsoft/bf-cli-config](https://github.com/microsoft/botframework-cli/tree/master/packages/config/src/commands/config/set/qnamaker.ts)_

## `bf config:set:telemetry`

啟用或停用可用來改善產品的匿名資料收集。 (命令群組呼叫和旗標使用情形)

```text
USAGE
  $ bf config:set:telemetry

OPTIONS
  -d, --disable  Disable tlemetry
  -e, --enable   Enable tlemetry
  -h, --help     config:set:telemetry help
```

_參閱程式碼：[@microsoft/bf-cli-config](https://github.com/microsoft/botframework-cli/tree/master/packages/config/src/commands/config/set/telemetry.ts)_

## `bf config:show`

顯示組態檔

```text
USAGE
  $ bf config:show

OPTIONS
  -h, --help  config:show help
```

_參閱程式碼：[@microsoft/bf-cli-config](https://github.com/microsoft/botframework-cli/tree/master/packages/config/src/commands/config/show.ts)_

## `bf config:show:qnamaker`

顯示 QnA Maker 設定

```text
USAGE
  $ bf config:show:qnamaker

OPTIONS
  -h, --help  config:show:qnamaker help
```

_參閱程式碼：[@microsoft/bf-cli-config](https://github.com/microsoft/botframework-cli/tree/master/packages/config/src/commands/config/show/qnamaker.ts)_

## `bf config:show:telemetry`

顯示遙測設定

```text
USAGE
  $ bf config:show:telemetry

OPTIONS
  -h, --help  config:show:telemetry help
```

_參閱程式碼：[@microsoft/bf-cli-config](https://github.com/microsoft/botframework-cli/tree/master/packages/config/src/commands/config/show/telemetry.ts)_

## `bf help [COMMAND]`

顯示 bf 命令的說明

```text
USAGE
  $ bf help [COMMAND]

ARGUMENTS
  COMMAND  command to show help for

OPTIONS
  --all  see all commands in CLI
```

_參閱程式碼：[@oclif/plugin-help](https://github.com/oclif/plugin-help/blob/v2.1.6/src/commands/help.ts)_

## `bf luis`

轉換、轉譯 luis/lu 檔案或產生原始程式碼。

```text
USAGE
  $ bf luis

OPTIONS
  -h, --help  Display Luis available commands
```

_參閱程式碼：[@microsoft/bf-lu](https://github.com/microsoft/botframework-cli/tree/master/packages/lu/src/commands/luis/index.ts)_

## `bf luis:convert`

將 .lu 檔案轉換成 LUIS 應用程式 JSON 模型 (或反過來)

```text
USAGE
  $ bf luis:convert

OPTIONS
  -f, --force                    If --out flag is provided with the path to an existing file, overwrites that file
  -h, --help                     luis:convert help
  -i, --in=in                    Source .lu file(s) or LUIS application JSON model
  -o, --out=out                  Output file or folder name. If not specified stdout will be used as output
  -r, --recurse                  Indicates if sub-folders need to be considered to file .lu file(s)
  --culture=culture              Lang code for the LUIS application
  --description=description      Text describing the LUIS application
  --log                          Enables log messages
  --name=name                    Name of the LUIS application
  --schemaversion=schemaversion  Schema version of the LUIS application
  --sort                         When set, intent, utterances, entities are alphabetically sorted in .lu files
  --versionid=versionid          Version ID of the LUIS application
```

_參閱程式碼：[@microsoft/bf-lu](https://github.com/microsoft/botframework-cli/tree/master/packages/lu/src/commands/luis/convert.ts)_

## `bf luis:generate:cs`

從匯出的 (JSON) LUIS 模型產生強型別的 C# 原始程式碼。

```text
USAGE
  $ bf luis:generate:cs

OPTIONS
  -f, --force            If --out flag is provided with the path to an existing file, overwrites that file
  -h, --help             luis:generate:cs help
  -i, --in=in            Path to the file containing the LUIS application JSON model
  -o, --out=out          Output file or folder name. If not specified stdout will be used as output
  --className=className  Name of the autogenerated class (can include namespace)
```

_參閱程式碼：[@microsoft/bf-lu](https://github.com/microsoft/botframework-cli/tree/master/packages/lu/src/commands/luis/generate/cs.ts)_

## `bf luis:generate:ts`

從匯出的 (JSON) LUIS 模型產生強型別的 typescript 原始程式碼。

```text
USAGE
  $ bf luis:generate:ts

OPTIONS
  -f, --force            If --out flag is provided with the path to an existing file, overwrites that file
  -h, --help             luis:generate:ts help
  -i, --in=in            Path to the file containing the LUIS application JSON model
  -o, --out=out          Output file or folder name. If not specified stdout will be used as output
  --className=className  Name of the autogenerated class
```

_參閱程式碼：[@microsoft/bf-lu](https://github.com/microsoft/botframework-cli/tree/master/packages/lu/src/commands/luis/generate/ts.ts)_

## `bf luis:translate`

轉譯指定的 LUIS 應用程式 JSON 模型或 .lu 檔案

```text
USAGE
  $ bf luis:translate

OPTIONS
  -f, --force                  If --out flag is provided with the path to an existing file, overwrites that file
  -h, --help                   luis:translate help
  -i, --in=in                  Source .lu file(s) or LUIS application JSON model
  -o, --out=out                Output folder name. If not specified stdout will be used as output
  -r, --recurse                Indicates if sub-folders need to be considered to file .lu file(s)
  --srclang=srclang            Source lang code. Auto detect if missing.
  --tgtlang=tgtlang            (required) Comma separated list of target languages.
  --translate_comments         When set, machine translate comments found in .lu file
  --translate_link_text        When set, machine translate link description in .lu file
  --translatekey=translatekey  (required) Machine translation endpoint key.
```

_參閱程式碼：[@microsoft/bf-lu](https://github.com/microsoft/botframework-cli/tree/master/packages/lu/src/commands/luis/translate.ts)_

## `bf qnamaker`

QnA Maker

```text
USAGE
  $ bf qnamaker

OPTIONS
  -h, --help  Display QnA Maker CLI available commands
```

_參閱程式碼：[@microsoft/bf-qnamaker](https://github.com/microsoft/botframework-cli/tree/master/packages/qnamaker/src/commands/qnamaker/index.ts)_

## `bf qnamaker:alterations`

用來取代和列出變更的命令

```text
USAGE
  $ bf qnamaker:alterations

OPTIONS
  -h, --help  display qnamaker:alterations available commands
```

_參閱程式碼：[@microsoft/bf-qnamaker](https://github.com/microsoft/botframework-cli/tree/master/packages/qnamaker/src/commands/qnamaker/alterations/index.ts)_

## `bf qnamaker:alterations:list`

下載使用者已新增的所有文字變更 (同義字)。

```text
USAGE
  $ bf qnamaker:alterations:list

OPTIONS
  -h, --help                         qnamaker:alterations:list command help
  --endpoint=endpoint                Overrides public endpoint https://westus.api.cognitive.microsoft.com/qnamaker/v4.0/

  --subscriptionKey=subscriptionKey  Specifies the qnamaker Ocp-Apim-Subscription Key (found in Keys under Resource
                                     Management section for your Qna Maker cognitive service). Overrides the
                                     subscriptionkey value present in the config
```

_參閱程式碼：[@microsoft/bf-qnamaker](https://github.com/microsoft/botframework-cli/tree/master/packages/qnamaker/src/commands/qnamaker/alterations/list.ts)_

## `bf qnamaker:alterations:replace`

以提供的記錄取代 KB 的文字變更 (同義字)。

```text
USAGE
  $ bf qnamaker:alterations:replace

OPTIONS
  -h, --help                         qnamaker:alterations:replace command help
  -i, --in=in                        File path to the WordAlterationsDTO object to send in the body of the request
  --endpoint=endpoint                Overrides public endpoint https://westus.api.cognitive.microsoft.com/qnamaker/v4.0/

  --subscriptionKey=subscriptionKey  Specifies the qnamaker Ocp-Apim-Subscription Key (found in Keys under Resource
                                     Management section for your Qna Maker cognitive service). Overrides the
                                     subscriptionkey value present in the config
```

_參閱程式碼：[@microsoft/bf-qnamaker](https://github.com/microsoft/botframework-cli/tree/master/packages/qnamaker/src/commands/qnamaker/alterations/replace.ts)_

## `bf qnamaker:convert`

將 .lu 檔案轉換成 QnA 應用程式 JSON 模型 (或反過來)。

```text
USAGE
  $ bf qnamaker:convert

OPTIONS
  -f, --force    If --out flag is provided with the path to an existing file, overwrites that file
  -h, --help     qnamaker:convert help
  -i, --in=in    Source .qna file(s) or QnA KB JSON file
  -o, --out=out  Output file or folder name. If not specified stdout will be used as output
  -r, --recurse  Indicates if sub-folders need to be considered to file .qna file(s)
  --alterations  Indicates if files is QnA Alterations
  --log          Enables log messages
  --name=name    Name of the QnA KB
  --sort         When set, questions collections are alphabetically sorted are alphabetically sorted in .qna files
```

_參閱程式碼：[@microsoft/bf-lu](https://github.com/microsoft/botframework-cli/tree/master/packages/lu/src/commands/qnamaker/convert.ts)_

## `bf qnamaker:endpointkeys`

用於重新整理和列出金鑰的命令

```text
USAGE
  $ bf qnamaker:endpointkeys

OPTIONS
  -h, --help  display qnamaker:endpointkeys available commands
```

_參閱程式碼：[@microsoft/bf-qnamaker](https://github.com/microsoft/botframework-cli/tree/master/packages/qnamaker/src/commands/qnamaker/endpointkeys/index.ts)_

## `bf qnamaker:endpointkeys:list`

列出所有目前可用來查詢您私用端點的有效 endpointKeys

```text
USAGE
  $ bf qnamaker:endpointkeys:list

OPTIONS
  -h, --help                         qnamaker:endpointkeys:list command help
  --endpoint=endpoint                Overrides public endpoint https://westus.api.cognitive.microsoft.com/qnamaker/v4.0/

  --subscriptionKey=subscriptionKey  Specifies the qnamaker Ocp-Apim-Subscription Key (found in Keys under Resource
                                     Management section for your Qna Maker cognitive service). Overrides the
                                     subscriptionkey value present in the config
```

_參閱程式碼：[@microsoft/bf-qnamaker](https://github.com/microsoft/botframework-cli/tree/master/packages/qnamaker/src/commands/qnamaker/endpointkeys/list.ts)_

## `bf qnamaker:endpointkeys:refresh`

當您懷疑金鑰已遭盜用時，重新產生端點金鑰

```text
USAGE
  $ bf qnamaker:endpointkeys:refresh

OPTIONS
  -h, --help                         qnamaker:endpoints:refresh command help
  --endpoint=endpoint                Overrides public endpoint https://westus.api.cognitive.microsoft.com/qnamaker/v4.0/
  --keyType=keyType                  (required) Type of Key. (PrimaryKey/SecondaryKey)

  --subscriptionKey=subscriptionKey  Specifies the qnamaker Ocp-Apim-Subscription Key (found in Keys under Resource
                                     Management section for your Qna Maker cognitive service). Overrides the
                                     subscriptionkey value present in the config
```

_參閱程式碼：[@microsoft/bf-qnamaker](https://github.com/microsoft/botframework-cli/tree/master/packages/qnamaker/src/commands/qnamaker/endpointkeys/refresh.ts)_

## `bf qnamaker:endpointsettings`

用來取得和更新端點設定的命令

```text
USAGE
  $ bf qnamaker:endpointsettings

OPTIONS
  -h, --help  display qnamaker:update available commands
```

_參閱程式碼：[@microsoft/bf-qnamaker](https://github.com/microsoft/botframework-cli/tree/master/packages/qnamaker/src/commands/qnamaker/endpointsettings/index.ts)_

## `bf qnamaker:endpointsettings:get`

取得端點的端點設定。

```text
USAGE
  $ bf qnamaker:endpointsettings:get

OPTIONS
  -h, --help                         qnamaker:endpointsettings:get command help
  --endpoint=endpoint                Overrides public endpoint https://westus.api.cognitive.microsoft.com/qnamaker/v4.0/
  --kbId=kbId                        Knowledgebase id to get metadata.

  --subscriptionKey=subscriptionKey  Specifies the qnamaker Ocp-Apim-Subscription Key (found in Keys under Resource
                                     Management section for your Qna Maker cognitive service). Overrides the
                                     subscriptionkey value present in the config
```

_參閱程式碼：[@microsoft/bf-qnamaker](https://github.com/microsoft/botframework-cli/tree/master/packages/qnamaker/src/commands/qnamaker/endpointsettings/get.ts)_

## `bf qnamaker:endpointsettings:update`

更新端點的端點設定。

```text
USAGE
  $ bf qnamaker:endpointsettings:update

OPTIONS
  -h, --help                         qnamaker:endpointsettings:update command help
  --activelearning                   Enable active learning. Disables if flag not set
  --endpoint=endpoint                Overrides public endpoint https://westus.api.cognitive.microsoft.com/qnamaker/v4.0/

  --subscriptionKey=subscriptionKey  Specifies the qnamaker Ocp-Apim-Subscription Key (found in Keys under Resource
                                     Management section for your Qna Maker cognitive service). Overrides the
                                     subscriptionkey value present in the config
```

_參閱程式碼：[@microsoft/bf-qnamaker](https://github.com/microsoft/botframework-cli/tree/master/packages/qnamaker/src/commands/qnamaker/endpointsettings/update.ts)_

## `bf qnamaker:init`

使用設定初始化組態檔。

```text
USAGE
  $ bf qnamaker:init

OPTIONS
  -h, --help           qnamaker:init command help
  --endpoint=endpoint  Overrides public endpoint https://westus.api.cognitive.microsoft.com/qnamaker/v4.0/
```

_參閱程式碼：[@microsoft/bf-qnamaker](https://github.com/microsoft/botframework-cli/tree/master/packages/qnamaker/src/commands/qnamaker/init.ts)_

## `bf qnamaker:kb`

用於操作知識庫的命令

```text
USAGE
  $ bf qnamaker:kb

OPTIONS
  -h, --help  display qnamaker:kb available commands
```

_參閱程式碼：[@microsoft/bf-qnamaker](https://github.com/microsoft/botframework-cli/tree/master/packages/qnamaker/src/commands/qnamaker/kb/index.ts)_

## `bf qnamaker:kb:create`

建立新的知識庫

```text
USAGE
  $ bf qnamaker:kb:create

OPTIONS
  -h, --help                         qnamaker:kb:create command help
  -i, --in=in                        File path to the CreateKbDTO object to send in the body of the request.
  --endpoint=endpoint                Overrides public endpoint https://westus.api.cognitive.microsoft.com/qnamaker/v4.0/

  --name=name                        Name of the kb you want to create. This will override the name of KB that might be
                                     present in the CreateKb DTO

  --save                             Save the kbId in config.

  --subscriptionKey=subscriptionKey  Specifies the qnamaker Ocp-Apim-Subscription Key (found in Keys under Resource
                                     Management section for your Qna Maker cognitive service). Overrides the
                                     subscriptionkey value present in the config
```

_參閱程式碼：[@microsoft/bf-qnamaker](https://github.com/microsoft/botframework-cli/tree/master/packages/qnamaker/src/commands/qnamaker/kb/create.ts)_

## `bf qnamaker:kb:delete`

依識別碼刪除知識庫

```text
USAGE
  $ bf qnamaker:kb:delete

OPTIONS
  -h, --help                         qnamaker:kb:delete command help
  --endpoint=endpoint                Overrides public endpoint https://westus.api.cognitive.microsoft.com/qnamaker/v4.0/
  --force                            Do not prompt for confirmation, force the operation

  --kbId=kbId                        Knowledgebase id to be deleted. Overrides the knowledge base id present in the
                                     config

  --subscriptionKey=subscriptionKey  Specifies the qnamaker Ocp-Apim-Subscription Key (found in Keys under Resource
                                     Management section for your Qna Maker cognitive service). Overrides the
                                     subscriptionkey value present in the config
```

_參閱程式碼：[@microsoft/bf-qnamaker](https://github.com/microsoft/botframework-cli/tree/master/packages/qnamaker/src/commands/qnamaker/kb/delete.ts)_

## `bf qnamaker:kb:export`

將知識庫 JSON 回應給 stdout

```text
USAGE
  $ bf qnamaker:kb:export

OPTIONS
  -h, --help                         qnamaker:kb:export command help
  --endpoint=endpoint                Overrides public endpoint https://westus.api.cognitive.microsoft.com/qnamaker/v4.0/
  --environment=environment          (required) Specifies whether environment is Test or Prod.

  --kbId=kbId                        Knowledgebase id to be exported. Overrides the knowledge base id present in the
                                     config

  --subscriptionKey=subscriptionKey  Specifies the qnamaker Ocp-Apim-Subscription Key (found in Keys under Resource
                                     Management section for your Qna Maker cognitive service). Overrides the
                                     subscriptionkey value present in the config
```

_參閱程式碼：[@microsoft/bf-qnamaker](https://github.com/microsoft/botframework-cli/tree/master/packages/qnamaker/src/commands/qnamaker/kb/export.ts)_

## `bf qnamaker:kb:get`

取得知識庫的相關中繼資料

```text
USAGE
  $ bf qnamaker:kb:get

OPTIONS
  -h, --help                         qnamaker:kb:get command help
  --endpoint=endpoint                Overrides public endpoint https://westus.api.cognitive.microsoft.com/qnamaker/v4.0/

  --kbId=kbId                        Knowledgebase id to get metadata. Overrides the knowledge base id present in the
                                     config

  --subscriptionKey=subscriptionKey  Specifies the qnamaker Ocp-Apim-Subscription Key (found in Keys under Resource
                                     Management section for your Qna Maker cognitive service). Overrides the
                                     subscriptionkey value present in the config
```

_參閱程式碼：[@microsoft/bf-qnamaker](https://github.com/microsoft/botframework-cli/tree/master/packages/qnamaker/src/commands/qnamaker/kb/get.ts)_

## `bf qnamaker:kb:list`

列出您所有的知識庫

```text
USAGE
  $ bf qnamaker:kb:list

OPTIONS
  -h, --help                         qnamaker:kb:list command help
  --endpoint=endpoint                Overrides public endpoint https://westus.api.cognitive.microsoft.com/qnamaker/v4.0/

  --subscriptionKey=subscriptionKey  Specifies the qnamaker Ocp-Apim-Subscription Key (found in Keys under Resource
                                     Management section for your Qna Maker cognitive service). Overrides the
                                     subscriptionkey value present in the config
```

_參閱程式碼：[@microsoft/bf-qnamaker](https://github.com/microsoft/botframework-cli/tree/master/packages/qnamaker/src/commands/qnamaker/kb/list.ts)_

## `bf qnamaker:kb:publish`

將知識庫中所有未發佈的項目發佈到生產端點。

```text
USAGE
  $ bf qnamaker:kb:publish

OPTIONS
  -h, --help                         qnamaker:kb:publish command help
  --endpoint=endpoint                Overrides public endpoint https://westus.api.cognitive.microsoft.com/qnamaker/v4.0/
  --kbId=kbId                        Knowledgebase id to pubish. Overrides the knowledge base id present in the config

  --subscriptionKey=subscriptionKey  Specifies the qnamaker Ocp-Apim-Subscription Key (found in Keys under Resource
                                     Management section for your Qna Maker cognitive service). Overrides the
                                     subscriptionkey value present in the config
```

_參閱程式碼：[@microsoft/bf-qnamaker](https://github.com/microsoft/botframework-cli/tree/master/packages/qnamaker/src/commands/qnamaker/kb/publish.ts)_

## `bf qnamaker:kb:replace`

以新內容取代知識庫內容

```text
USAGE
  $ bf qnamaker:kb:replace

OPTIONS
  -h, --help                         qnamaker:kb:replace command help
  -i, --in=in                        File path to the ReplaceKbDTO object to send in the body of the request
  --endpoint=endpoint                Overrides public endpoint https://westus.api.cognitive.microsoft.com/qnamaker/v4.0/
  --kbId=kbId                        Knowledgebase id. Overrides the knowledge base id present in the config

  --subscriptionKey=subscriptionKey  Specifies the qnamaker Ocp-Apim-Subscription Key (found in Keys under Resource
                                     Management section for your Qna Maker cognitive service). Overrides the
                                     subscriptionkey value present in the config
```

_參閱程式碼：[@microsoft/bf-qnamaker](https://github.com/microsoft/botframework-cli/tree/master/packages/qnamaker/src/commands/qnamaker/kb/replace.ts)_

## `bf qnamaker:kb:update`

新增或刪除現有知識庫的問與答配對和/或 URL

```text
USAGE
  $ bf qnamaker:kb:update

OPTIONS
  -h, --help                         qnamaker:kb:update command help

  -i, --in=in                        The file path to the UpdateKbOperationDTO object to send in the body of the
                                     request.

  --endpoint=endpoint                Overrides public endpoint https://westus.api.cognitive.microsoft.com/qnamaker/v4.0/

  --kbId=kbId                        Knowledgebase id. Overrides the knowledge base id present in the config

  --subscriptionKey=subscriptionKey  Specifies the qnamaker Ocp-Apim-Subscription Key (found in Keys under Resource
                                     Management section for your Qna Maker cognitive service). Overrides the
                                     subscriptionkey value present in the config

  --wait                             Wait for the operation to complete.
```

_參閱程式碼：[@microsoft/bf-qnamaker](https://github.com/microsoft/botframework-cli/tree/master/packages/qnamaker/src/commands/qnamaker/kb/update.ts)_

## `bf qnamaker:operationdetails`

取得作業詳細資料的命令

```text
USAGE
  $ bf qnamaker:operationdetails

OPTIONS
  -h, --help  display qnamaker:operationdetails available commands
```

_參閱程式碼：[@microsoft/bf-qnamaker](https://github.com/microsoft/botframework-cli/tree/master/packages/qnamaker/src/commands/qnamaker/operationdetails/index.ts)_

## `bf qnamaker:operationdetails:get`

取得特定長時間執行作業的詳細資料。

```text
USAGE
  $ bf qnamaker:operationdetails:get

OPTIONS
  -h, --help                         qnamaker:operationdetails:get command help
  --endpoint=endpoint                Overrides public endpoint https://westus.api.cognitive.microsoft.com/qnamaker/v4.0/
  --operationId=operationId          (required) Operation id.

  --subscriptionKey=subscriptionKey  Specifies the qnamaker Ocp-Apim-Subscription Key (found in Keys under Resource
                                     Management section for your Qna Maker cognitive service). Overrides the
                                     subscriptionkey value present in the config
```

_參閱程式碼：[@microsoft/bf-qnamaker](https://github.com/microsoft/botframework-cli/tree/master/packages/qnamaker/src/commands/qnamaker/operationdetails/get.ts)_

## `bf qnamaker:query`

查詢知識庫中的答案

```text
USAGE
  $ bf qnamaker:query

OPTIONS
  -h, --help                       qnamaker:query command help
  --context=context                Path to Context object json file with previous QnA

  --endpointKey=endpointKey        Specifies the endpoint key for your private QnA service (From qnamaker.ai portal user
                                   settings page). Overrides the value present in config

  --hostname=hostname              Specifies the url for your private QnA service. Overrides the value present in config

  --kbId=kbId                      Specifies the active qnamaker knowledgebase id. Overrides the value present in the
                                   config

  --qnaId=qnaId                    Exact qnaId to fetch from the knowledgebase, this field takes priority over question

  --question=question              (required) Query to get a prediction for

  --scorethreshold=scorethreshold  Specifies the confidence score threshold for the returned answer.

  --strictfilters=strictfilters    Path to json file with MetadataDTO[] e.g {"strictfilters": MetadataDTO[]}

  --test                           Query against the test index

  --top=top                        Specifies the number of matching results
```

_參閱程式碼：[@microsoft/bf-qnamaker](https://github.com/microsoft/botframework-cli/tree/master/packages/qnamaker/src/commands/qnamaker/query.ts)_

## `bf qnamaker:train`

將建議新增至知識庫。

```text
USAGE
  $ bf qnamaker:train

OPTIONS
  -h, --help                         qnamaker:get:kb command help

  --endpointKey=endpointKey          Specifies the endpoint key for your private QnA service.(from qnamaker.ai portal
                                     user settings page). Overrides the value present in config.

  --hostname=hostname                Specifies the url for your private QnA service. Overrides the value present in
                                     config.

  --in=in                            File path to the FeedbackRecordDTO object to send in the body of the request.

  --kbId=kbId                        Specifies the active qnamaker knowledgebase id. Overrides the value present in the
                                     config

  --subscriptionKey=subscriptionKey  Specifies the qnamaker Ocp-Apim-Subscription Key (found in Keys under Resource
                                     Management section for your Qna Maker cognitive service). Overrides the
                                     subscriptionkey value present in config
```

_參閱程式碼：[@microsoft/bf-qnamaker](https://github.com/microsoft/botframework-cli/tree/master/packages/qnamaker/src/commands/qnamaker/train.ts)_

## `bf qnamaker:translate`

轉譯指定的 QnA Maker 應用程式 JSON 模型或 .qna 檔案

```text
USAGE
  $ bf qnamaker:translate

OPTIONS
  -f, --force                  If --out flag is provided with the path to an existing file, overwrites that file
  -h, --help                   qnamaker:translate help
  -i, --in=in                  Source .qna file(s) or QnA maker application JSON model
  -o, --out=out                Output folder name. If not specified stdout will be used as output
  -r, --recurse                Indicates if sub-folders need to be considered to find .qna file(s)
  --srclang=srclang            Source lang code. Auto detect if missing.
  --tgtlang=tgtlang            (required) Comma separated list of target languages.
  --translate_comments         When set, machine translate comments found in .qna file
  --translate_link_text        When set, machine translate link description in .qna file
  --translatekey=translatekey  (required) Machine translation endpoint key.
```

_參閱程式碼：[@microsoft/bf-lu](https://github.com/microsoft/botframework-cli/tree/master/packages/lu/src/commands/qnamaker/translate.ts)_
<!-- commandsstop -->
