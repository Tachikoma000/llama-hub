# Airbyte Stripe Loader

The Airbyte Stripe Loader allows you to access different Stripe objects.

## Installation

* Install llama_hub: `pip install llama_hub`
* Install the stripe source: `pip install airbyte-source-stripe`

## Usage

Here's an example usage of the AirbyteStripeReader.

```python
from llama_hub.airbyte_stripe.base import AirbyteStripeReader

stripe_config = {
    # ...
}
reader = AirbyteStripeReader(config=stripe_config)
documents = reader.load_data(stream_name="invoices")
```

## Configuration

Check out the [Airbyte documentation page](https://docs.airbyte.com/integrations/sources/stripe/) for details about how to configure the reader.
The JSON schema the config object should adhere to can be found on Github: [https://github.com/airbytehq/airbyte/blob/master/airbyte-integrations/connectors/source-stripe/source_stripe/spec.yaml](https://github.com/airbytehq/airbyte/blob/master/airbyte-integrations/connectors/source-stripe/source_stripe/spec.yaml).

The general shape looks like this:
```python

{
  "client_secret": "<secret key>",
  "account_id": "<account id>",
  "start_date": "<date from which to start retrieving records from in ISO format, e.g. 2020-10-20T00:00:00Z>",
}
```

By default all fields are stored as metadata in the documents and the text is set to the JSON representation of all the fields. Construct the text of the document by passing a `record_handler` to the reader:
```python
def handle_record(record, id):
    return Document(doc_id=id, text=record.data["title"], extra_info=record.data)

reader = AirbyteStripeReader(config=stripe_config, record_handler=handle_record)
```

## Lazy loads

The `reader.load_data` endpoint will collect all documents and return them as a list. If there are a large number of documents, this can cause issues. By using `reader.lazy_load_data` instead, an iterator is returned which can be consumed document by document without the need to keep all documents in memory.

## Incremental loads

This loader supports loading data incrementally (only returning documents that weren't loaded last time or got updated in the meantime):
```python

reader = AirbyteStripeReader(config={...})
documents = reader.load_data(stream_name="invoices")
current_state = reader.last_state # can be pickled away or stored otherwise

updated_documents = reader.load_data(stream_name="invoices", state=current_state) # only loads documents that were updated since last time
```

This loader is designed to be used as a way to load data into [LlamaIndex](https://github.com/jerryjliu/gpt_index/tree/main/gpt_index) and/or subsequently used as a Tool in a [LangChain](https://github.com/hwchase17/langchain) Agent. See [here](https://github.com/emptycrown/llama-hub/tree/main) for examples.
