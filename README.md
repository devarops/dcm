# Data Control Manager

_dcm_ es una aplicación que emula control de versiones para archivos binarios de datos. Este _post_ presenta una historia de usuario de _dcm_.

## Crear repo _dcm_ y agregar archivos
Primero inicializamos un repo _dcm_ en la carpeta `datos`.

```
> cd datos
> dcm init
```

Vemos el estado del repo:

```
> dcm status
    ?   foo.bin
    ?   bar.bin
```

Agregamos los archivos de datos **binarios** al repo _dcm_:

```
> dcm add
```
Subimos los datos:
```
> dcm push
    pushing...
    done
```
## Consignar y subir cambios
El analista modificó `foo.bar`:
```
> dcm status
    M   foo.bin
```
Consignamos los cambios:
```
> dcm commit
```
Subimos la versión actual:
```
> dcm push
    pushing...
    done
```
## Descargar cambios
Para descargar actualizaciones en los archivos de datos
```
> dcm pull
    pulling...
    done
```
La instrucción anterior descarga la versión más nueva sin modificar tu copia local. Para actualizar tus archivos de datos:
```
> dcm update
```
## Actualiza sólo un archivo
Los comandos `pull` y `update` aceptan la opcion `--filename` para indicar el nombre del archivo que se desea altualizar
```
> dcm pull --filename foo.bin
    pulling...
    done
> dcm update --filename foo.bin
```
También aceptan el _checksum_ de la versión deseada de un archivo:
```
> dcm pull --checksum 4d186321c1a7f0f354b297e8914ab240
    pulling...
    done
> dcm update --checksum 4d186321c1a7f0f354b297e8914ab240
```

## ¿Cómo funciona?

**Crear repo _dcm_ y agregar archivos**

El comando `init` crea un archivo `dcm.json` y una carpeta `.dcm`.

Ejemplo de `dcm.json`:
``` json
{
    "resources": null
}
```
El directorio contiene dos achivos: `foo.bin` y `bar.bin`. El comando `status` busca en `dcm.json` el nombre de los archivos que encuentra en el directorio (y que no se encuentran en `.dcmignore`). Dado que `dcm.json` está vació, `status` despliega:
```
> dcm status
    ?   foo.bin
    ?   bar.bin
```
El comando `add` agrega a `dcm.json` el nombre y _checksum_ de los archivos marcados con `?`.
``` json
{
    "resources":
        [
            {
                "filename": "foo.bin",
                "checksum": "4D186321C1A7F0F354B297E8914AB240",
                "hash": "md5"
            },
            {
                "filename": "bar.bin",
                "checksum": "DD485E41F1758DEF296E1BC7377F8EA7",
                "hash": "md5"
            }
        ]
}
```


> Debes consignar en tu sistema de control de versiones (hg o git) los cambios en `dcm.json` cada vez que uses el comando `dcm add`.

Además, `add` copia los archivos `foo.bin` y `bar.bin` a la carpeta `.dcm/` renombrándolos con el checksum:

```
> ls .dcm/
    md5_4D186321C1A7F0F354B297E8914AB240.dcm
    md5_DD485E41F1758DEF296E1BC7377F8EA7.dcm
```

> Ignora en tu sistema de control de versiones (hg o git) la carpeta `.dcm`.

El comando `push` sube los archivos `md5_*.dcm` a un servidor _FTP_.

**Consignar y subir cambios**

Cuando se modifica `foo.bar`, el comando `status` encuentra que el checksum actual de `foo.bar` no coincide con el checksum guardado en el archivo `dcm.json` y despliega:
```
> dcm status
    M   foo.bin
```
El comando `commit` actualiza `dcm.json` con el nuevo checksum:
``` json
{
    "resources":
        [
            {
                "filename": "foo.bin",
                "checksum": "md5_0AD066A5D29F3F2A2A1C7C17DD082A79",
                "hash": "md5"
            },
            {
                "filename": "bar.bin",
                "checksum": "DD485E41F1758DEF296E1BC7377F8EA7",
                "hash": "md5"
            }
        ]
}
```

> Debes consignar en tu sistema de control de versiones (hg o git) los cambios en `dcm.json` cada vez que uses el comando `dcm commit`.

y hace una copia de `foo.bin` a la carpeta `.dcm/` renombrándola con el nuevo checksum:

```
> ls .dcm/
    md5_4D186321C1A7F0F354B297E8914AB240.dcm
    md5_DD485E41F1758DEF296E1BC7377F8EA7.dcm
    md5_0AD066A5D29F3F2A2A1C7C17DD082A79.dcm
```

Vemos que en la carpeta `.dcm/` hay tres archivos: dos versiones de `foo.bin` y una versión de `bar.bin`. El nuevo archivo `md5_0AD066A5D29F3F2A2A1C7C17DD082A79.dcm` todavía no se ha subido al servidor. Para subir la nueva versión de `foo.bar` usamos nuevamente el comando `push`.

**Descargar cambios**

El comando `pull` obtiene una lista de los checksum guardados en `dcm.json`, construye una lista de nombres de archivo con el formato `.dcm/md5_{checksum}.dcm`, y descarga del servidor FTP los archivos que no se encuentren en la carpeta `.dcm/`.

El comando `update` sobreescribe tus archivos de datos binarios con los archivos `.dcm/md5_{checksum}.dcm` usando la relación `filename`- `checksum` descrita por `dcm.json`.

**Actualiza sólo un archivo**
Las opciones `--filename` y `--checksum` de los comandos `add`, `commit`, `pull` y `update` usan los campos correspondientes en `dcm.json` para realizar la operación sobre sólo un archivo.
