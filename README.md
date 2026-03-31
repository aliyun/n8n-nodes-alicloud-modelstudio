# n8n-nodes-alicloud-modelstudio

This is an n8n community node. It lets you use Alibaba Cloud Model Studio in your n8n workflows.

Alibaba Cloud Model Studio is a unified API platform for Alibaba Cloud's Qwen, Wan, and Z-Image models, providing access to language, vision, image generation, and video generation capabilities through a single service.

[Installation](#installation)  
[Quickstart](#quickstart)  
[Operations](#operations)  
[Credentials](#credentials)  
[Compatibility](#compatibility)  
[Usage](#usage)  
[Resources](#resources)  

## Installation

Follow the [installation guide](https://docs.n8n.io/integrations/community-nodes/installation/) in the n8n community nodes documentation.

## Quickstart

### 1. Install the node

In your n8n instance go to **Settings → Community Nodes → Install** and enter:

```
n8n-nodes-alicloud-modelstudio
```

### 2. Add credentials

1. Sign up for an [Alibaba Cloud](https://www.alibabacloud.com/) account if you don't have one.
2. Open the [Model Studio console](https://modelstudio.console.alibabacloud.com), click on the gear icon, navigate to **API Key**, and create a new key.
3. In n8n go to **Credentials → New**, search for **Alibaba Cloud Model Studio API**, and paste the key.

### 3. Build your first workflow

**Chat with a Qwen model**

1. Add an **Alibaba Cloud Model Studio** node to your canvas.
2. Select credential created in step 2.
3. Set **Resource** → `Text` and **Operation** → `Message`.
4. Add a message with **Role** `User` and type your question in **Content**.
5. Execute the node. The result field `text` contains the model's reply.

**Generate an image**

1. Set **Resource** → `Image` and **Operation** → `Generate`.
2. Type a description in **Prompt** (e.g. `A futuristic city at sunset`).
3. Execute the node. The result field contains the image URL.
4. Chain an **Image → Download** operation (or a separate node execution) to save the image as a binary file.

**Generate a video from text**

1. Set **Resource** → `Video` and **Operation** → `Text to Video`.
2. Type a description in **Prompt** (e.g. `A cat playing with a ball of yarn`).
3. Choose **Resolution** and **Duration**, then execute.
4. The node polls until the video is ready and returns the URL in `videoUrl`.
5. Chain a **Video → Download** operation to save the MP4 as a binary file.

## Operations

### Text

#### Message

Send a chat completion request to a Qwen language model.

| Field | Description |
|---|---|
| **Model** | Language model to use (see table below). |
| **Messages** | One or more messages that form the conversation. Each message has a **Role** (`User`, `System`, `Assistant`) and **Content**. |
| **Simplify Output** | Return only the generated `text` string. Disable to include token usage and the full response. |


**Options**

| Option | Range / Type | Description |
|---|---|---|
| Enable Search | boolean | Allow the model to query the web for up-to-date information. |
| Max Tokens | integer ≥ 1 | Maximum number of tokens to generate. Default `2000`. |
| Repetition Penalty | 1.0 – 2.0 | Penalises repeated tokens. Higher = less repetition. Default `1.1`. |
| Seed | integer | Fixed seed for reproducible outputs. |
| Stop Sequences | comma-separated strings | The model stops generating when it produces one of these sequences. |
| Temperature | 0 – 2 | Sampling randomness. Lower = more deterministic. Default `1`. |
| Top K | 1 – 100 | Limits sampling pool to the top K tokens. Default `50`. |
| Top P | 0 – 1 | Nucleus sampling threshold. Default `0.9`. |

---

### Image

#### Analyze

Ask a question about an image using a Qwen vision-language (VL) model.

| Field | Description |
|---|---|
| **Model** | `Qwen-VL Flash` (fast) or `Qwen-VL Plus` (enhanced). |
| **Image URL** | Publicly accessible URL of the image to analyse. |
| **Question** | Text question or instruction about the image (e.g. `What is in this image?`). |
| **Simplify Output** | Return only the `text` answer. Disable to include usage data and the full response. |

**Options**

| Option | Range / Type | Description |
|---|---|---|
| Temperature | 0 – 2 | Sampling randomness. Default `1`. |
| Max Tokens | integer ≥ 1 | Maximum tokens in the answer. Default `2000`. |

#### Generate

Generate an image from a text prompt.

| Field | Description |
|---|---|
| **Model** | Image generation model (see table below). |
| **Prompt** | Text description of the image to generate. |
| **Simplify Output** | Return only the image URL. Disable to include the full API response. |

**Available models**

| Name | Model ID | Notes |
|---|---|---|
| Z-Image Turbo | `z-image-turbo` | Fast generation (default) |
| Wan 2.6 T2I | `wan2.6-t2i` | Wanx text-to-image model |
| Qwen Image | `qwen-image` | Standard Qwen image model |
| Qwen Image Plus | `qwen-image-plus` | Enhanced Qwen image model |
| Qwen Image Max | `qwen-image-max` | Most capable Qwen image model |

**Options**

| Option | Applies to | Description |
|---|---|---|
| Size | `z-image-turbo`, `wan2.6-t2i` | `1024×1024`, `720×1280`, or `1280×720`. Default `1024×1024`. |
| Size | `qwen-image*` | `1328×1328 (1:1)`, `1472×1104 (4:3)`, `1104×1472 (3:4)`, `1664×928 (16:9)`, `928×1664 (9:16)`. Default `1664×928`. |
| Prompt Extend | all | Automatically expand and enhance the prompt before generation. |

#### Download

Download a previously generated image from its URL and return it as a binary file.

| Field | Description |
|---|---|
| **URL** | The image URL returned by a Generate operation (e.g. an `oss-cn-beijing.aliyuncs.com` URL). |
| **Put Output in Field** | Name of the binary output field. Default `data`. |

The node returns the image as binary data (JPEG or PNG detected from the `Content-Type` header) and sets the standard n8n binary metadata fields.

---

### Video

#### Text to Video

Generate a video from a text prompt using the Wan 2.6 T2V model. The operation is asynchronous — the node automatically polls the task until it completes and returns the result.

| Field | Description |
|---|---|
| **Model** | `wan2.6-t2v` (only available model). |
| **Prompt** | Text description of the video to generate. |
| **Resolution** | `720P` or `1080P`. Default `1080P`. |
| **Duration (seconds)** | Video length between 2 and 15 seconds. Default `5`. |
| **Shot Type** | `Single` (one continuous shot) or `Multi` (multi-shot narrative). Default `Single`. |
| **Simplify Output** | Return only `videoUrl`. Disable to include task ID, usage data, and the full response. |

**Options**

| Option | Description |
|---|---|
| Prompt Extend | Automatically expand and enhance the prompt before generation. |
| Audio | Generate audio for the video. Enabled by default. |
| Audio URL | Public URL to an audio file (MP3, WAV, etc.) to use as the video soundtrack instead of AI-generated audio. |

#### Image to Video

Animate a still image by providing a first-frame image URL and an optional motion prompt. The operation is asynchronous — the node polls until the task completes.

| Field | Description |
|---|---|
| **Model** | `wan2.6-i2v-flash` (fast, default) or `wan2.6-i2v` (standard quality). |
| **Image URL** | Publicly accessible URL of the still image to use as the first frame. |
| **Prompt** | Optional text description of the desired motion and visual characteristics. |
| **Resolution** | `720P` or `1080P`. Default `1080P`. |
| **Duration (seconds)** | Video length between 2 and 15 seconds. Default `5`. |
| **Shot Type** | `Single` (one continuous shot) or `Multi` (multi-shot narrative). Default `Single`. |
| **Simplify Output** | Return only `videoUrl`. Disable to include task ID, usage data, and the full response. |

**Options**

| Option | Description |
|---|---|
| Prompt Extend | Automatically expand and enhance the prompt before generation. |
| Audio | Generate audio for the video. Enabled by default. |
| Audio URL | Public URL to an audio file to use as the video soundtrack. |

#### Download

Download a previously generated video from its URL and return it as a binary MP4 file.

| Field | Description |
|---|---|
| **URL** | The video URL returned by a Text to Video or Image to Video operation. |
| **Put Output in Field** | Name of the binary output field. Default `data`. |

The node returns the video as binary data with `video/mp4` content type and sets the standard n8n binary metadata fields.

## Credentials

1. Sign up for an [Alibaba Cloud](https://www.alibabacloud.com/) account if you don't have one.
2. Open the [Model Studio console](https://modelstudio.console.alibabacloud.com), click on the gear icon, navigate to **API Key**, and create a new key.
3. In n8n go to **Credentials → New**, search for **Alibaba Cloud Model Studio API**, and paste the key.

## Compatibility

Tested with n8n version 2.8.3.

## Usage

Select a **Resource** (`Text`, `Image`, or `Video`) from the dropdown in the node, then choose an **Operation**.

**Simplify Output** is available on every operation. When enabled, the node returns only the essential result — generated text, an image URL, or a video URL. When disabled, the full API response including token usage statistics is returned.

**Text → Message:** Add one or more messages to build the conversation. A single `User` message is enough for simple prompts. Use a `System` message to set model behaviour. Enable the **Enable Search** option to let the model fetch up-to-date information from the web.

**Image → Generate + Download:** Generate returns a temporary hosted URL. Use the **Image → Download** operation (or a second node) with that URL to save the image as a binary file for further processing or storage.

**Video → Text to Video / Image to Video:** These are async operations. The node submits the generation task and then polls the status endpoint automatically — no manual polling node is needed. Generation can take 30 seconds to several minutes depending on duration and resolution. Once complete, the video URL is returned in `videoUrl`. Chain a **Video → Download** operation to persist the MP4.

## Resources

* [n8n community nodes documentation](https://docs.n8n.io/integrations/#community-nodes)
* [Alibaba Cloud Model Studio documentation](https://www.alibabacloud.com/help/en/model-studio/)
* [Model Studio API reference](https://www.alibabacloud.com/help/en/model-studio/model-api-reference)
