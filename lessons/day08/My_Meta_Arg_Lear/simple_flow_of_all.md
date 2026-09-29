Count:
------
      count
        ↓
      number
        ↓
      count.index
        ↓
      aws_instance.app[0]

For_each + set
----------------

    for_each + set
      ↓
    value
      ↓
    each.value
      ↓
    aws_instance.app["payment"]

For_each + Map
--------------

    for_each + map
      ↓
    key + value
      ↓
    each.key / each.value

For_each + map(objects)
-----------------------

    for_each + map(object)
      ↓
    key + object
      ↓
    each.key / each.value.attribute
