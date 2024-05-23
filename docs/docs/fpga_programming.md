# Programmation FPGA depuis Linux

## Comment générer un fichier BIN

!!! warning "NE PAS UTILISER VIVADO !"
    Vivado génère des fichiers BIN qui ne sont pas compatibles avec le chargement du FPGA depuis Linux.
    
    ```plaintext	
    Paramètres -> Bitstream -> -bin_file : activer
    ```

    Le fichier sera généré dans `proj_name/proj_name.runs/impl_1` ne sera pas compatible avec le chargement du FPGA depuis Linux.
    
    l'utilisation de `bootgen` est nécessaire pour générer un fichier BIN compatible.


1. Créez un fichier de configuration BIF nommé avec le contenu suivant :

  ```plaintext
  all:
  {
      project.bit /* Nom du fichier Bitstream */
  }
  ```

2. Utilisez la commande `bootgen` pour générer le fichier BIN :

  ```bash
  bootgen -image project.bif -arch zynq -process_bitstream bin
  ```

## Comment utiliser ce fichier BIN ?

1. Connectez-vous en tant qu'utilisateur root :

  ```bash
  sudo -i
  ```

2. Définissez les flags pour le Full Bitstream :

  ```bash
  echo 0 > /sys/class/fpga_manager/fpga0/flags
  ```

3. Chargez le Bitstream dans le PL :

   ```bash
   mkdir -p /lib/firmware
   cp project.bit.bin /lib/firmware/
   echo project.bit.bin > /sys/class/fpga_manager/fpga0/firmware
   ```

4. Félicitations, vous avez terminé !