# OP_RETURN-size

Descargue los archivos fuente de Bitcoin clonando el repositorio con git.
```
git clone https://github.com/bitcoin/bitcoin.git
```
Berkeley DB (BDB) solo es necesario para la compatibilidad o prueba de billeteras heredadas. Tenga en cuenta que puede deshabilitar el soporte de billetera heredada pasando ``` --sin-bdb ``` a sus opciones ``` ./configure ``` a continuación.

- Para instalar BDB 4.8, una versión heredada compatible con versiones anteriores, ejecute ``` brew install berkeley-db@4 ``` para macOS y, para Linux, consulte la documentación de Bitcoin Core build-*.md para su sistema operativo (el script heredado install_db4.sh se eliminó en Bitcoin Core v25 con PR 26834).
- Si ya tiene instalada otra versión de BDB que desea usar, simplemente agregue ``` --with-incompatible-bdb ``` a sus indicadores ``` ./configure ``` a continuación. Se recomienda esta opción a menos que necesite específicamente BDB 4.8.

Ahora desde la carpeta Bitcoin compile Bitcoin desde la fuente.

![2023-03-05_19-35](https://user-images.githubusercontent.com/127045720/222979199-af94a94b-4a93-40f7-b292-1641df9d294e.png)


```
 ./autogen.sh![imagen](https://user-images.githubusercontent.com/127045720/222985072-9c506edb-99e2-4bd0-b779-a4b2f5580f2d.png)

 
 ./configure --with-incompatible-bdb --enable-suppress-external-warnings 
```

![2023-03-05_19-38](https://user-images.githubusercontent.com/127045720/222979334-9eb2234c-4c75-40f9-b977-3416e3822012.png)

make, o si tiene varios núcleos de CPU, que es el caso habitual hoy en día, puede decirle a make que los use todos y reducir significativamente el tiempo de compilación con

```
make -j "$(($(nproc) + 1))"
```

![2023-03-05_20-09](https://user-images.githubusercontent.com/127045720/222980660-b7d78bbd-b9bc-4175-b0ca-f36ac643c2bd.png)




Finalmente, instale todo el código compilado usando el comando make install como se muestra a continuación.


```
sudo make install
```
![2023-03-05_20-11](https://user-images.githubusercontent.com/127045720/222980775-b5b5bfa2-8565-4ea9-8679-135d2ae20382.png)

Para comprobar la isntalacion puede ejecutar ``` bitcoin-cli --version ```

![2023-03-05_20-13](https://user-images.githubusercontent.com/127045720/222980855-450fdb2c-f128-4546-b1bc-bc5da28c6614.png)


Una vez todo instalado nos movemos al directorio script/ en el cual esta el archivo standard.h en el cual cambiaremos el tamaño maximo del OP_RETURN

![2023-03-05_20-17](https://user-images.githubusercontent.com/127045720/222981043-a574d770-6eda-4db6-910c-e5750a902054.png)

Editamos el archivo con nano y buscamos la linea 39 como se ve en la siguiente imagen


![2023-03-05_20-22](https://user-images.githubusercontent.com/127045720/222981244-ac4fe68d-f239-4b79-8699-662c848e9d9a.png)


```
nano standard.h
```

A cotinuacion cambiamos el valor por defecto al que nos parezca conveniente en este caso 390000 bytes


![2023-03-05_20-26](https://user-images.githubusercontent.com/127045720/222981437-03314db1-857a-4460-9ec6-d8d4222c60dc.png)

Ahora volvemos a compilar el codigo con el cambio guardado

```
make -j "$(($(nproc)+1))"
```
e instalamos nuevamente bitcoincli con el comando ``` sudo make install ``` 

Para ejecutar Bitcoincli en modo regtest pasamos el comando ``` bitcoin-qt -regtest ```
Saldra una ventana como la de la imagen

![2023-03-05_20-55](https://user-images.githubusercontent.com/127045720/222982649-13c0a710-7abc-47ed-ac2b-03afe144aaa4.png)

Ahora pasamos el comando ``` bitcoin-cli -regtest settxfee 0.001 ``` para subir los fees y poder hacer una transaccion de prueba
![2023-03-05_20-58](https://user-images.githubusercontent.com/127045720/222982807-808f0eb3-adeb-4289-966b-d79f615f4e29.png)

Una vez echo esto vamos a enviar una transaccion de prueba con un OP_RETURN mayor de 80 bytes para esto hacemos los siguientes pasos:


Crea la variable ```op_return_data``` como se ve en la siguiente imagen

![2023-03-05_21-49](https://user-images.githubusercontent.com/127045720/222985147-6125c9d0-b52a-4656-96c0-99a70d8441b5.png)


Luego selecione un identificador de una moneda no gastada y la salva en la variable ```utxo_txid```

![2023-03-05_21-13](https://user-images.githubusercontent.com/127045720/222983489-b5f15046-780a-438e-a18f-b18e98232965.png)

```
utxo_txid=$(bitcoin-cli -regtest listunspent | jq -r '.[1] | .txid') 
utxo_vout=$(bitcoin-cli -regtest listunspent | jq -r '.[1] | .vout')
changeaddress=$(bitcoin-cli -regtest getrawchangeaddress)
rawtxhex=$(bitcoin-cli -regtest -named createrawtransaction inputs='''[ { "txid": "'$utxo_txid'", "vout": '$utxo_vout' } ]''' outputs='''{ "data": "'$op_return_data'", "'$changeaddress'": 0.0146 }''')
bitcoin-cli -regtest decoderawtransaction $rawtxhex 
```


Una ves echo estos pasos tendremos una transaccion con el OP_RETURN de la variable 


![2023-03-05_21-31](https://user-images.githubusercontent.com/127045720/222984358-cf8b4f2a-3c0c-4ec7-a3f5-46767499c077.png)

Ahora firmamos la transaccion 
```
bitcoin-cli -regtest signrawtransactionwithwallet $rawtxhex

```
y la enviamos

```
bitcoin-cli -regtest -named sendrawtransaction hexstring=020000000001014d832fb5b9a5788b0ddbf4ca5561efb0c4697a5a37d91395cf29a96c157fecc80000000000fdffffff020000000000000000fdb7016a4db301796f7563616e6f6e6c7973746f72653830627974657361746174696d65796f7563616e6f6e6c7973746f72653830627974657361746174696d65796f7563616e6f6e6c7973746f72653830627974657361746174696d65796f7563616e6f6e6c7973746f72653830627974657361746174696d65796f7563616e6f6e6c7973746f72653830627974657361746174696d65796f7563616e6f6e6c7973746f72653830627974657361746174696d65796f7563616e6f6e6c7973746f72653830627974657361746174696d65796f7563616e6f6e6c7973746f72653830627974657361746174696d65796f7563616e6f6e6c7973746f72653830627974657361746174696d65796f7563616e6f6e6c7973746f72653830627974657361746174696d65796f7563616e6f6e6c7973746f72653830627974657361746174696d65796f7563616e6f6e6c7973746f72653830627974657361746174696d65796f7563616e6f6e6c7973746f72653830627974657361746174696d65796f7563616e6f6e6c7973746f72653830627974657361746174696d65796f7563616e6f6e6c7973746f72653830627974657361746174696d6520471600000000001600148025bed7d36235d0c59c76cac35c5840e92d2af3024730440220138ec889180d5470e4770b0d54c531802f688abfd673660a0ec06a649d48eff202200a937f0b8464003aa2c516d8fd40d311cc023426b52a0603ea54b5af2171b86a012103d3e59d2925d80323e8ef5405792b104da8a61e2bbfae3f77bab8a2d2641874dc00000000 maxfeerate=0

544bf01df9353755cc91fffb89e04ee379f63bf9e19e52ccf8772a0e8f3fd771
```
Con btc-explorer podemo ver que el OP_RETURN supera los 80 bytes

![2023-03-05_21-42](https://user-images.githubusercontent.com/127045720/222984769-6ddb5a8e-e6ef-4cd2-81c6-ab0eaa392e13.png)
















