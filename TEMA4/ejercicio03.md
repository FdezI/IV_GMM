# Ejercicios 3:
### Crear imágenes con estos formatos (y otros que se encuentren tales como VMDK) y manipularlas a base de montarlas o con cualquier otra utilidad que se encuentre

Vamos a probar los diferentes tipos de imágenes que soporta [QEMU](http://en.wikibooks.org/wiki/QEMU/Images), todos pueden ser generados con **qemu-img** siguiendo el patrón `qemu-img create -f FORMATO NOMBRE_ARCHIVO TAMAÑO`, por ejemplo:

* **raw**: qemu-img create -f raw imagen-raw.img 100M
* **qcow2**: qemu-img create -f qcow2 imagen-qcow2.qcow2 100M
* **vmdk**: qemu-img create -f vmdk imagen-vmdk.vmdk 100M
* **vdi**: qemu-img create -f vdi imagen-vdi.vdi 100M

![eje03_img01](imagenes/eje03_img01.png)

Como estás imagenes están recien creados y en blanco, no tienen ningún formato de sistema de archivos, por lo que si intentamos montarlas obtendremos un error.

Montar la imagen como un dispositivo **loop**:

```
sudo mount -o loop,offset=32256 imagen-raw.img /mnt/monta1
```

Montar la imagen como un dispositivo **NBD**:

```
sudo modprobe nbd max_part=16
sudo qemu-nbd -c /dev/nbd0 imagen-qcow2.qcow2
sudo partprobe /dev/nbd0
sudo mount /dev/nbd0 /mnt/image
```

![eje03_img02](imagenes/eje03_img02.png)

Para poder montarlos deberemos primero convertir los archivos de imagen en dispositivos **loop** (`sudo losetup -v -f imagen-raw.img`) y seguidamente darle un formato de sistema de archivos cualquiera como puede ser **ext4** (`sudo mkfs.ext4 /dev/loop0`):

![eje03_img03](imagenes/eje03_img03.png)

Ahora ya podremos montar la imagen como el dispositivo **loop** correspondiente (`sudo mount /dev/loop0 /mnt/monta1/`), comprobando que se ha montado correctamente (`df -h`):

![eje03_img04](imagenes/eje03_img04.png)

Como el formato **vdi** es el formato usado para los discos duros virtuales de **VirtualBox**, podemos usar dichas imagenes para añadirlas como nuevas unidades de disco a máquinas virtuales que tengamos creadas en **VirtualBox** (aunque también podriamos usar para este mismo programa imagenes **vmdk**, el formato de imagen típico de VMware). Por ejemplo, voy a usar una máquina virtual que tengo creada con Windows XP; nos situamos en la configuración de la máquina virtual, en el apartado **Almacenamiento**:

![eje03_img05](imagenes/eje03_img05.png)

Pulsamos el **botón más a la derecha** que está en línea con **"Controller: IDE"** para agregar un **nuevo disco duro virtual**, como queremos usar un archivo de imagen, seleccionamos el botón de la opción **"Seleccionar disco existente**:

![eje03_img06](imagenes/eje03_img06.png)

Veremos como se ha añadido una nueva unidad de disco duro. En su información podemos ver que se trata de un disco duro que usa **"Almacenamiento reservado dinámicamente** (ejemplo de provisionamiento delgado), siendo su **tamaño actual 1 KB** y su **tamaño virtual 100 MB**; además, su ubicación es nuestro archivo de imagen:

![eje03_img07](imagenes/eje03_img07.png)

Si arrancamos la máquina virtual, nos reconocerá una nueva unidad de disco duro, pero sin inicializar ni formato alguno:

![eje03_img08](imagenes/eje03_img08.png)

Así que lo inicializamos y le damos formato para que se reconozca como un disco duro usable:

![eje03_img09](imagenes/eje03_img09.png)

Y ya podemos usarlo en nuestra máquina virtual como otra unidad de disco duro más:

![eje03_img10](imagenes/eje03_img10.png)
