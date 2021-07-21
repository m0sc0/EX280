# Certs

openssl x509 -noout -in ../26-05-2021/apps-ocp-term.crt -text 
## Gen a private key
openssl genrsa -out training.key 2048
## Gen a certificate request
openssl req -new -key training.key -out training.csr -subj "XXXX"
## Sign csr and outoput crt
openssl x509 -req -in training.csr -out training.crt -passin file=passphrase.txt -CA training-CA.pem -CAkey training-CA.key -CAcreateserial -ext training.ext -sha256 -days 350

 
