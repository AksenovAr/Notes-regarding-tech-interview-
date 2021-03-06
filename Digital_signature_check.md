### Here I give example how to check digital signature using the follow algorythms SHA384withRSA, SHA512withRSA, SHA384withECDSA, SHA512withECDSA
I used openssl library for those purposes.


### Type value can be NID_sha384 or NID_sha512
### Source code example 
```
bool verify_RSAmessage(unsigned char const* pDigest, int digestLen,
                            unsigned char const* pSignature, int sigLen, int type)
{

    if (sPubKey.empty())
    {
        _ERROR("Public key is empty.");
        return false;
    }

    BIO *pBio = BIO_new_mem_buf( (void*) sPubKey.c_str(), sPubKey.length());
    if (pBio == nullptr)
    {
        _ERROR("Create BIO error.");
        return false;
    }

    RSA *pRsaPubKey = nullptr;
    if (sPubKey.find("-----BEGIN PUBLIC KEY-----") != std::string::npos)
    {
        pRsaPubKey = PEM_read_bio_RSA_PUBKEY(pBio, NULL, NULL, NULL);
        if (pRsaPubKey == nullptr)
        {
            BIO_free_all(pBio);
            _ERROR("Public key file reading error.");
            return false;
        }
    }
    else if (sPubKey.find("-----BEGIN RSA PUBLIC KEY-----") != std::string::npos)
    {
        pRsaPubKey = PEM_read_bio_RSAPublicKey(pBio, NULL, NULL, NULL);
        if (pRsaPubKey == nullptr)
        {
            BIO_free_all(pBio);
            _ERROR("Public key PEM RSA format reading error.");
            return false;
        }
    }
    else
    {
        BIO_free_all(pBio);
        _ERROR("Not suitable format of public key file.");
        return false;
    }

    int state = RSA_verify(type, pDigest, digestLen, pSignature, sigLen, pRsaPubKey);
    LOGP_INFO("Verify RSA message result is %d.", state);

    RSA_free(pRsaPubKey);
    BIO_free_all(pBio);
    return (state == 1) ? true : false;
}
```

### Type value can be SHA384withECDSA or SHA512withECDSA
### Source code example 

```
bool verify_ECDSAmessage(unsigned char const* pMessage, int msgLen,
                            unsigned char const* pSignature, int sigLen, int type)
{
    if (sPubKey.empty())
    {
        _ERROR("Public key is empty.");
        return false;
    }

    EC_KEY *pEcPubKey = nullptr;

    BIO *pBio = BIO_new_mem_buf( (void*) sPubKey.c_str(), sPubKey.length());
    if (pBio == nullptr)
    {
        _ERROR("Create BIO error.");
        return false;
    }

    pEcPubKey = PEM_read_bio_EC_PUBKEY(pBio, NULL, NULL, NULL);
    if (pEcPubKey == nullptr)
    {
        BIO_free_all(pBio);

        _ERROR("Public key PEM format reading error.");
        return false;
    }

    int state = ECDSA_verify(type, pMessage, msgLen, pSignature, sigLen, pEcPubKey);
    _INFO("Verify ECDSA message result is %d.", state);

    BIO_free_all(pBio);
    EC_KEY_free(pEcPubKey);
    return (state == 1) ? true : false;
}
```

### Before using functions we should init some values

 ```
switch (algorithm)
   {
       case SHA384withRSA:
       {
           unsigned char digest[SHA384_DIGEST_LENGTH];
           SHA512_CTX ctx;
           SHA384_Init(&ctx);
           SHA384_Update(&ctx, pMessage, msgLen);
           SHA384_Final(digest, &ctx);

           res = _verify_RSAmessage(digest, sizeof(digest), pSignature, sigLen, NID_sha384);
           break;
       }
       case SHA512withRSA:
       {
           unsigned char digest[SHA512_DIGEST_LENGTH];
           SHA512_CTX ctx;
           SHA512_Init(&ctx);
           SHA512_Update(&ctx, pMessage, msgLen);
           SHA512_Final(digest, &ctx);

           res = _verify_RSAmessage(digest, sizeof(digest), pSignature, sigLen, NID_sha512);
           break;
       }
       case SHA384withECDSA:
       {
            unsigned char digest[SHA384_DIGEST_LENGTH];
            SHA512_CTX ctx;
            SHA384_Init(&ctx);
            SHA384_Update(&ctx, pMessage, msgLen);
            SHA384_Final(digest, &ctx);

            res = _verify_ECDSAmessage(digest, sizeof(digest), pSignature, sigLen, NID_secp384r1);
            break;
       }
       case SHA512withECDSA:
       {
            unsigned char digest[SHA512_DIGEST_LENGTH];
            SHA512_CTX ctx;
            SHA512_Init(&ctx);
            SHA512_Update(&ctx, pMessage, msgLen);
            SHA512_Final(digest, &ctx);

            res = _verify_ECDSAmessage(digest, sizeof(digest), pSignature, sigLen, NID_secp521r1);
            break;
       }
   }
```
### Very importante notice !

Don`t forget read public key in sPubKey std::string value


to be continue
