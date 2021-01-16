# 4. Mapping & Analysis

[Topics up to here covered things already known.]

## Reindex documents
Reindex API:
```json
POST /_reindex
{
    "source": {
        "index": "my_ind"
    },
    "dest": {
        "index": "new_ind"
    },
    "script": {
        "source": """
            [script goes here]
        """
    }
}
```
Field mappings can't be deleted. By reindexing, you can remove fields by not including them.

## Multifield mappings
Can add multiple mappings to a field, such as a text and keyword mapping to a text field. It can be useful to be able to select by keywords e.g. for use in Kibana visuals. (I've also added this for Kibana by setting `"fielddata": True`).  
You can even set different analysers for a single field.

## Index templates
