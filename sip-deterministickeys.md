```
Title: Deterministic Bucket and File Keys
Author: Chris Pollard <chris@storj.io>
Status: Draft
Type: Standard
Created: 2016-10-21
```

Abstract
--------

Core-cli feature which will generate encryption keys based on a master starting seed, allowing users to easily transfer keys to another device, backup their present and future keys offline, and share encryption keys either on a per-bucket or per-file basis.

Motivation
----------

Currently, all keys are generated randomly on a per-file basis and stored encrypted in the user's keyring. This does not allow any easy way for a user to syncronize keys across devices. To do this currently, it would require either trusting the bridge with keyring, creating direct connections between devices, or introducing a new asymmetric encryption scheme. None of these are easily implemented.

This proposal allows users to syncronize keys simply by typing a twelve word mnemonic into each new device. This seed will allow users to generate all of their keys deterministically for any future bucket or file.

Secondly, there is no way to make a future-proof offline backup. A backup storage device would have to constantly be updated with the latest version of the keyring. This could compromise the backup's security.

Finally, users might want to grant other users access to all the files in a bucket. Currently, this would require all file keys to be sent to the target, which would be difficult to manage. This proposal introduces bucket keys which can be used to deterministically calculate all of the file keys in a bucket. A user could share this bucket key with another user so that they can encrypt or decrypt any file in a bucket, or share it with the bridge to make an entire bucket public.

Specification
-------------

Generate a long password in the form of a twelve word mnemonic taken from 2048 possible words. This acts as a long, secure password. This mnemonic will be encrypted into the keyring using the keyring's standard method of encryption. The bucket key is generated by taking the sha512 digest of the concatenation of the mnemonic and bucket id. The file key is generated similarly using the bucketKey and fileId.

```javascript
DeterministicKeyIv.getDeterministicKey = function(key, id){
  var buffer = utils.sha512(key + id);
  return buffer.toString('hex').substring(0, 64);
};
```

### Integration

Upon starting, core-cli will check if an encrypted deterministic seed exists in the keyring. If it does not, it will prompt the user either to generate one or import one. If they choose to generate one, it will print out a twelve word mnemonic and store this encrypted in the keyring. If the user chooses to import one, he will have to enter an existing twelve word mnemonic. There will also be two new core-cli commands to print and delete the mnemonic.