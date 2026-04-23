# Файловые системы и LVM-1
## LVM - начало работы
т.к. работа требует везде права root : ` sudo su `
- Смотрим испытательный стенд 
```
lslbk
```
<details>
<summary>доступные устройтсва</summary>
<img width="800" height="166" alt="image" src="https://github.com/user-attachments/assets/9193e99f-d549-406a-b1a2-1c9a1a241600" />
</details>

```
lvmdiskscan
```
<details>
<summary>вывод</summary>
<img width="800" height="165" alt="image" src="https://github.com/user-attachments/assets/1ab2814f-6b34-4860-a702-775f8941d032" />
</details>

- Создаем PV  ` pvcreate /dev/sdb `
  получаем
  <img width="800" height="40" alt="image" src="https://github.com/user-attachments/assets/4c91f1f8-1307-4b14-9980-a93f6fe76e60" /><br>
- Создаем VG ` vgcreate lvm_hw /dev/sdb `
  <img width="800" height="40" alt="image" src="https://github.com/user-attachments/assets/b3bfda65-c212-403d-944e-8beccc774859" /><br>
- Создаем LV `lvcreate -l+80%FREE -n test lvm_hw`
  <img width="800" height="40" alt="image" src="https://github.com/user-attachments/assets/3f68d49d-2a42-41e1-a1f1-63c0baee43c1" /><br>
- посмотрим только что созданый VG `vgdisplay lvm_hw`
 <details>
<summary>вывод</summary>
<img width="800" height="400" alt="image" src="https://github.com/user-attachments/assets/db9264e6-9405-4a0d-a1dd-d70a8c8db743" />
</details>

- просмотр о дисках которые включены в VG ` vgdisplay -v lvm_hw | grep 'PV Name' `
  <img width="800" height="35" alt="image" src="https://github.com/user-attachments/assets/2776b126-a906-43f0-95a8-4f86b1ad3c26" />
- просмотр детальной информации
  <details>
<summary>vgs</summary>

```
  VG     #PV #LV #SN Attr   VSize   VFree
  lvm_hw   1   1   0 wz--n- <10.00g 2.00g
```
</details>
или
<details>
<summary>lvs</summary>

```
  LV   VG     Attr       LSize  Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  test lvm_hw -wi-a----- <8.00g
```
</details>

- займем свободный кусочек новым LV задав размер абсолютной величиной `lvcreate -L100M -n small lvm_hw`
  
  ```
  Logical volume "small" created.
  ```
- проверим текущее состояние `lvs`
  <img width="800" height="53" alt="image" src="https://github.com/user-attachments/assets/7ad335c8-e266-4fb7-97ea-24dbe4f8aa1a" />
- Создадим на LV файловую систему и смонтируем его
  
```
mkfs.ext4 /dev/lvm_hw/test
```
<details>
<summary>результат</summary>
<img width="800" height="164" alt="image" src="https://github.com/user-attachments/assets/87d34c69-56ec-415b-85f4-113915b32903" />
</details>

- создаем путь для монтирования и проверяем
  1) `mkdir -p /mnt/lvm_hw/data`
  2) `mount /dev/lvm_hw/test /mnt/lvm_hw/data`
  3) `mount | grep /mnt`
  результат 
  <img width="800" height="81" alt="image" src="https://github.com/user-attachments/assets/a527bcd5-62ec-49fe-9959-98d5af02e679" />

## Расширение LVM
- создаем PV для нового диска `pvcreate /dev/sdc`
  `Physical volume "/dev/sdc" successfully created.`
- расширяем VG на включив его в существующий `vgextend lvm_hw /dev/sdc`
  `Volume group "lvm_hw" successfully extended`
- проверим что всё прошло успешно и новый VG где нужно `vgdisplay -v lvm_hw | grep 'PV Name'`
  <img width="800" height="49" alt="image" src="https://github.com/user-attachments/assets/907d2080-717c-4465-bb53-01698da3ec7c" />
- удостоверимся что места стало больше `vgs`
  <img width="800" height="48" alt="image" src="https://github.com/user-attachments/assets/3c1252ad-e41c-479b-9ee3-960947fa9b45" />
- Имитируем заполнение места
  ```
  dd if=/dev/zero of=/mnt/lvm_hw/data/test.log bs=1M count=8000 status=progress
  ```
  получаем 
 


- заейм часть нового свободного места `lvextend -l+80%FREE /dev/lvm_hw/test`


