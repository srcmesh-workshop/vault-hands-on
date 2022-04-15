# Identity-based Access

## Enable Auth Methods

此處我們用 `userpass` 作為認證方式

```bash
$ vault auth enable -path="<MOUNT_PATH>" userpass

# Example
$ vault auth enable -path="user-0" userpass
```

* `MOUNT_PATH`: 為此認證方式掛載的目錄位置 (須為小寫)

確認是否成功，並取得 `Accessor` 以供下面指令使用

```bash
$ vault auth list -format=json | jq -r '.["<MOUNT_PATH>/"].accessor' > accessor.txt

# Example
$ vault auth list -format=json | jq -r '.["user-0/"].accessor' > accessor.txt
```

## Create a user

```bash
$ vault write auth/<MOUNT_PATH>/users/<USERNAME> password="<PASSWORD>"

# Example
$ vault write auth/user-0/users/user-0 password="user-0"
```

## Create an entity

```bash
$ vault write -format=json identity/entity name="<USERNAME>" \
     metadata=<KEY>="<VALUE>" \
     | jq -r ".data.id" > entity_id.txt

# Example
$ vault write -format=json identity/entity name="user-0-entity" \
     metadata=organization="Srcmesh" \
     metadata=team="DEV" \
     | jq -r ".data.id" > entity_id.txt
```

## Bind user to entity 

```bash
$ vault write identity/entity-alias name="<USERNAME>" \
     canonical_id=<ENTITY_ID> \
     mount_accessor=<ACCESSOR> \
     custom_metadata=<KEY>=<VALUE>
     
# Example
$ vault write identity/entity-alias name="user-0" \
     canonical_id=$(cat entity_id.txt) \
     mount_accessor=$(cat accessor.txt) \
     custom_metadata=account="Tester"
```

## View entity details

```bash
$ vault read -format=json identity/entity/id/$(cat entity_id.txt) | jq -r ".data"
```

應該看到類似的輸出

```json
{
  "aliases": [
    {
      "canonical_id": "ffa714d9-f188-cb0f-4d51-9136e098104f",
      "creation_time": "2022-04-12T09:19:25.837358Z",
      "custom_metadata": {
        "account": "Tester"
      },
      "id": "2db84461-3e01-8b8e-a731-44147791941a",
      "last_update_time": "2022-04-12T09:19:25.837358Z",
      "local": false,
      "merged_from_canonical_ids": null,
      "metadata": null,
      "mount_accessor": "auth_userpass_608444e1",
      "mount_path": "auth/user-0/",
      "mount_type": "userpass",
      "name": "user-0"
    }
  ],
  "creation_time": "2022-04-12T09:11:35.187488Z",
  "direct_group_ids": [],
  "disabled": false,
  "group_ids": [],
  "id": "ffa714d9-f188-cb0f-4d51-9136e098104f",
  "inherited_group_ids": [],
  "last_update_time": "2022-04-12T09:11:35.187488Z",
  "merged_entity_ids": null,
  "metadata": {
    "organization": "Srcmesh",
    "team": "DEV"
  },
  "name": "user-0-entity",
  "namespace_id": "root",
  "policies": []
}
```

## Create a group

```bash
$ vault write identity/group name="<GROUP_NAME>" \
     member_entity_ids=<ENTITY_ID> \
     metadata=<KEY>=<VALUE> \
     | jq -r ".data.id" > group_id.txt

$ vault write -format=json identity/group name="engineers" \
     member_entity_ids=$(cat entity_id.txt) \
     metadata=region="Asia" \
     | jq -r ".data.id" > group_id.txt
```

## Reference

* https://learn.hashicorp.com/tutorials/vault/identity