# Local Transformers Embedding Client

The `TransformersEmbeddingClient` is a `EmbeddingClient` implementation that computes, locally, [sentence embeddings](https://www.sbert.net/examples/applications/computing-embeddings/README.html#sentence-embeddings-with-transformers) using a selected [sentence transformer](https://www.sbert.net/).

It uses [pre-trained](https://www.sbert.net/docs/pretrained_models.html) transformer models, serialized into the [Open Neural Network Exchange (ONNX)](https://onnx.ai/) format.

The [Deep Java Library](https://djl.ai/) and the Microsoft [ONNX Java Runtime](https://onnxruntime.ai/docs/get-started/with-java.html) libraries are applied to run the ONNX models and compute the embeddings in Java.

## Serialize the Tokenizer and the Transformer Model

To run things in Java, we need to serialize the Tokenizer and the Transformer Model into ONNX format.

### Serialize with optimum-cli

One, quick, way to achieve this, is to use the [optimum-cli](https://huggingface.co/docs/optimum/exporters/onnx/usage_guides/export_a_model#exporting-a-model-to-onnx-using-the-cli) command line tool.

Following snippet creates an python virtual environment, installs the required packages and runs the optimum-cli to serialize (e.g. export) the models:

```bash
python3 -m venv venv
source ./venv/bin/activate
(venv) pip install --upgrade pip
(venv) pip install optimum onnx onnxruntime
(venv) optimum-cli export onnx --model sentence-transformers/all-MiniLM-L6-v2 onnx-output-folder
```

The `optimum-cli` command exports the [sentence-transformers/all-MiniLM-L6-v2](https://huggingface.co/sentence-transformers/all-MiniLM-L6-v2) transformer into the `onnx-output-folder` folder. Later includes the `tokenizer.json` and `model.onnx` files used by the embedding client.

## Apply the ONNX model

Use the `setTokenizerResource(tokenizerJsonUri)` and `setModelResource(modelOnnxUri)` methods to set the URI locations of the exported `tokenizer.json` and `model.onnx` files.
The `classpath:`, `file:` or `https:` URI schemas are supported.

If no other model is explicitly set, the `TransformersEmbeddingClient` defaults to [sentence-transformers/all-MiniLM-L6-v2](https://huggingface.co/sentence-transformers/all-MiniLM-L6-v2) model:

|     |  |
| -------- | ------- |
| Dimensions  |384    |
| Avg. performance | 58.80     |
| Speed    | 14200 sentences/sec    |
| Size    | 80MB    |


Following snippet illustrates how to use the `TransformersEmbeddingClient`:

```java
TransformersEmbeddingClient embeddingClient = new TransformersEmbeddingClient();

// (optional) defaults to classpath:/onnx/all-MiniLM-L6-v2/tokenizer.json
embeddingClient.setTokenizerResource("classpath:/onnx/all-MiniLM-L6-v2/tokenizer.json");
// (optional) defaults to classpath:/onnx/all-MiniLM-L6-v2/model.onnx
embeddingClient.setModelResource("classpath:/onnx/all-MiniLM-L6-v2/model.onnx");

// (optional) defaults to ${java.io.tmpdir}/spring-ai-onnx-model
// Only the http/https resources are cached by default.
embeddingClient.setResourceCacheDirectory("/tmp/onnx-zoo");

embeddingClient.afterPropertiesSet();

List<List<Double>> embeddings =
	embeddingClient.embed(List.of("Hello world", "World is big"));

```








