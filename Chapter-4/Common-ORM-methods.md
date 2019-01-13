# 常用的的 ORM 方法

既然使用了 ORM 来定义数据模型，那么如何来操作数据呢？这就要用到 Odoo ORM 的方法，接下来介绍几个常用的 ORM 方法。  

在介绍这些方法之前，我们需要进入 Odoo 的交互式命令行，进入 odoo_dev 目录并输入以下命令：  

```shell
$ ./odoo-bin shell -c ./odoorc
```

💡 建议在使用 Odoo 交互式命令行之前安装 `IPython`，方便进行代码补全和使用。  
  
⚠️ 进入命令行前必须将配置文件的 `db_name` 参数配置成安装了 bangumi 应用相同的数据库，或使用 `-d` 参数指定数据库。  

在上一章节有提到 api 模块中的 Environment 类，我们就是要用这个初始化出一个 `env` 来操作数据库，不过 Odoo 已经在启动命令行时帮助我们将这个 env 初始化好了。  

```plain
In [1]: env                               
Out[1]: <odoo.api.Environment at 0x108d84438>
```

我们可以利用 env 来得到我们的数据库模型，并且操作数据库表。  

```plain
In [2]: env['bangumi.bangumi']
Out[2]: bangumi.bangumi()
```

接下来我们将利用以下常见的 ORM 方法来操作我们的数据模型。  

⚠️ 在 Odoo 交互式命令行中对数据进行，创建、修改和删除后必须要执行 `env.cr.commit()` 才会将数据操作写入数据库中。

* **create()**  
    
    `create` 方法用于创建模型数据，参数可以为 `dict` 或 `list` 类型，创建后会返回创建的记录或记录集。  
    
    ```plain
    In [3]: env['bangumi.bangumi'].create({'name': 'Fate', 'total': 24})                
    Out[3]: bangumi.bangumi(1,)
    In [4]: env['bangumi.bangumi'].create([{'name': 'SAO', 'total': 24}, {'name': 'Jojo', 'total': 24}])
    Out[4]: bangumi.bangumi(2, 3)
    In [5]: env.cr.commit()
    ```

* **search()**
    
    `search` 方法用于搜索已存在的数据，与大多数 ORM 框架不同，Odoo 搜索时传入的是一个 `domain`，它是一个列表形式的参数，在后面的章节会详细介绍这里先不做讲解。
    除了 domain 参数，还有 `offset`、`limit`、`order` 和 `count`，这些都是可选字段。
    
    ```plain
    In [6]: env['bangumi.bangumi'].search([('name', '=', 'Fate')]).name
    Out[6]: 'Fate'
    In [7]: env['bangumi.bangumi'].search([('total', '=', 24)], limit=2)
    Out[7]: bangumi.bangumi(1, 2)
    ```

* **write()**
    
    `write` 方法用于修改数据，传入参数为 `dict` 类型，修改后会返回布尔值 `True`。
    
    ```plain
    In [8]: env['bangumi.bangumi'].search([('name', '=', 'SAO')]).write({'total': 12})
    Out[8]: True
    In [9]: env.cr.commit()
    ```

* **browse()**
    
    `browse` 方法可以通过 `id` 直接返回数据或结果集，参数可以为 `dict` 或 `list` 类型。
    
    ```plain
    In [10]: env['bangumi.bangumi'].browse(1)
    Out[10]: bangumi.bangumi(1,)
    In [11]: records = env['bangumi.bangumi'].browse([1, 2, 3])
    In [12]: records                       
    Out[12]: bangumi.bangumi(1, 2, 3)
    ```

* **unlink()**
    
    `unlink` 方法用于用于删除数据或结果集，删除后返回布尔值 `True`，结果集可以为空，相当于没有操作。  
    
    ```plain
    In [13]: env['bangumi.bangumi'].search([{'name': 'Jojo'}]).unlink()
    Out[13]: True 
    In [14]: env.cr.commit()
    ```

* **exists()**

    `exists` 用于过滤出数据库中确实存在的数据，这个用法比较奇特需要注意一下。  
    
    在上文中我们用 browse 方法搜索出了记录集 `records`。  
    
    ```plain
    In [15]: records                       
    Out[15]: bangumi.bangumi(1, 2, 3)
    ```
    
    但是实际上 id 为`3`的数据已经被我们删除了，我们一般会觉得需要重新使用 `search` 方法搜索数据，这里我们可以利用 Odoo 的 exists 方法，他只会返回数据库中存在的数据或记录集。  
    
    ```plain
    In [16]: records.exists()
    Out[16]: bangumi.bangumi(1, 2,)
    ```
    
    利用这个函数我们可以在删除了一些数据后，用于判断数据或记录集是否为空。  
    
    ```python
    if not records.exists():
        # do something ...
        pass
    ```


