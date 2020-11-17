# gpg-all-i-know

All I know about using the awesome Gnu Privacy Guard 


## With the secrethub manager

Here is an example work I did for a company I worked for.

Used version og GPG : 

```bash 
$ gpg --version
gpg (GnuPG) 2.2.20
libgcrypt 1.8.6
Copyright (C) 2020 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <https://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Home: /home/jibl/.gnupg
Supported algorithms:
Pubkey: RSA, ELG, DSA, ECDH, ECDSA, EDDSA
Cipher: IDEA, 3DES, CAST5, BLOWFISH, AES, AES192, AES256, TWOFISH,
        CAMELLIA128, CAMELLIA192, CAMELLIA256
Hash: SHA1, RIPEMD160, SHA256, SHA384, SHA512, SHA224
Compression: Uncompressed, ZIP, ZLIB, BZIP2
$ date
Tue 17 Nov 2020 01:58:53 AM CET
```

```bash
# --- # --- # --- # --- # --- # --- # --- # --- # --- #
# --- # --- # --- # --- # --- # --- # --- # --- # --- #
# --- # --- # --- # --- # --- # --- # --- # --- # --- #
#        GPG Key Pair of the Gravitee Lab Bot         #
#                for Github SSH Service               #
#                to GPG sign maven artifacts          #
#        >>> GPG version 2.x ONLY!!!                  #
# --- # --- # --- # --- # --- # --- # --- # --- # --- #
# --- # --- # --- # --- # --- # --- # --- # --- # --- #
# --- # --- # --- # --- # --- # --- # --- # --- # --- #
# -------------------------------------------------------------- #
# -------------------------------------------------------------- #
# for the Gravitee CI CD Bot in
# the https://github.com/gravitee-lab Github Org
# -------------------------------------------------------------- #
# -------------------------------------------------------------- #
# https://www.gnupg.org/documentation/manuals/gnupg-devel/Unattended-GPG-key-generation.html
export GRAVITEEBOT_GPG_USER_NAME="Gravitee.io Lab Bot"
export GRAVITEEBOT_GPG_USER_NAME_COMMENT="Gravitee CI CD Bot in the https://github.com/gravitee-lab Github Org"
export GRAVITEEBOT_GPG_USER_EMAIL="contact@gravitee-lab.io"
export GRAVITEEBOT_GPG_PASSPHRASE="th3gr@vit331sdab@s3"

echo "Creating a GPG KEY Pair for the Gravitee.io bot"
echo "# -----------------------------------"
# https://www.gnupg.org/documentation/manuals/gnupg-devel/Unattended-GPG-key-generation.html
export GNUPGHOME="$(mktemp -d)"
cat >./gravitee-lab-cicd-bot.gpg <<EOF
%echo Generating a basic OpenPGP key
Key-Type: RSA
Key-Length: 4096
Subkey-Type: RSA
Subkey-Length: 4096
Name-Real: ${GRAVITEEBOT_GPG_USER_NAME}
Name-Comment: ${GRAVITEEBOT_GPG_USER_NAME_COMMENT}
Name-Email: ${GRAVITEEBOT_GPG_USER_EMAIL}
Expire-Date: 0
Passphrase: ${GRAVITEEBOT_GPG_PASSPHRASE}
# Do a commit here, so that we can later print "done" :-)
%commit
%echo done
EOF
gpg --batch --generate-key ./gravitee-lab-cicd-bot.gpg
echo "GNUPGHOME=[${GNUPGHOME}] remove that directory when finished initializing secrets"
ls -allh ${GNUPGHOME}
gpg --list-secret-keys
gpg --list-keys

export GRAVITEEBOT_GPG_SIGNING_KEY_ID=$(gpg --list-signatures -a "${GRAVITEEBOT_GPG_USER_NAME} (${GRAVITEEBOT_GPG_USER_NAME_COMMENT}) <${GRAVITEEBOT_GPG_USER_EMAIL}>" | grep 'sig' | tail -n 1 | awk '{print $2}')
echo "GRAVITEEBOT - GPG_SIGNING_KEY=[${GRAVITEEBOT_GPG_SIGNING_KEY_ID}]"

# ++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++ #
# ------------------------------------------------------------------------------------------------ #
# -- SAVING SECRETS TO SECRETHUB --                                                   -- SECRET -- #
# ------------------------------------------------------------------------------------------------ #
echo "To verify the GPG signature \"Somewhere else\" we will also need the GPG Public key"
export GPG_PUB_KEY_FILE="$(pwd)/graviteebot.gpg.pub.key"
export GPG_PRIVATE_KEY_FILE="$(pwd)/graviteebot.gpg.priv.key"

# --- #
# saving public and private GPG Keys to files
gpg --export -a "${GRAVITEEBOT_GPG_USER_NAME} (${GRAVITEEBOT_GPG_USER_NAME_COMMENT}) <${GRAVITEEBOT_GPG_USER_EMAIL}>" | tee ${GPG_PUB_KEY_FILE}
# gpg --export -a "Jean-Baptiste Lasselle <jean.baptiste.lasselle.pegasus@gmail.com>" | tee ${GPG_PUB_KEY_FILE}
# -- #
# Will be interactive for private key : you
# will have to type your GPG password
gpg --export-secret-key -a "${GRAVITEEBOT_GPG_USER_NAME} (${GRAVITEEBOT_GPG_USER_NAME_COMMENT}) <${GRAVITEEBOT_GPG_USER_EMAIL}>" | tee ${GPG_PRIVATE_KEY_FILE}
# gpg --export-secret-key -a "Jean-Baptiste Lasselle <jean.baptiste.lasselle.pegasus@gmail.com>" | tee ${GPG_PRIVATE_KEY_FILE}



export SECRETHUB_ORG="gravitee-lab"
export SECRETHUB_REPO="cicd"
secrethub mkdir --parents "${SECRETHUB_ORG}/${SECRETHUB_REPO}/graviteebot/gpg"

export GRAVITEEBOT_GPG_USER_NAME="Gravitee.io Lab Bot"
export GRAVITEEBOT_GPG_USER_NAME_COMMENT="Gravitee CI CD Bot in the https://github.com/gravitee-lab Github Org"
export GRAVITEEBOT_GPG_USER_EMAIL="contact@gravitee-lab.io"
export GRAVITEEBOT_GPG_PASSPHRASE="th3gr@vit331sdab@s3"


echo "${GRAVITEEBOT_GPG_USER_NAME}" | secrethub write "${SECRETHUB_ORG}/${SECRETHUB_REPO}/graviteebot/gpg/user_name"
echo "${GRAVITEEBOT_GPG_USER_NAME_COMMENT}" | secrethub write "${SECRETHUB_ORG}/${SECRETHUB_REPO}/graviteebot/gpg/user_name_comment"
echo "${GRAVITEEBOT_GPG_USER_EMAIL}" | secrethub write "${SECRETHUB_ORG}/${SECRETHUB_REPO}/graviteebot/gpg/user_email"
echo "${GRAVITEEBOT_GPG_PASSPHRASE}" | secrethub write "${SECRETHUB_ORG}/${SECRETHUB_REPO}/graviteebot/gpg/passphrase"
echo "${GRAVITEEBOT_GPG_SIGNING_KEY_ID}" | secrethub write "${SECRETHUB_ORG}/${SECRETHUB_REPO}/graviteebot/gpg/key_id"
secrethub write --in-file ${GPG_PUB_KEY_FILE} "${SECRETHUB_ORG}/${SECRETHUB_REPO}/graviteebot/gpg/pub_key"
secrethub write --in-file ${GPG_PRIVATE_KEY_FILE} "${SECRETHUB_ORG}/${SECRETHUB_REPO}/graviteebot/gpg/private_key"

# ++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++ #
# ------------------------------------------------------------------------------------------------ #
# -- MISSING !!!!! -- Restore GPG Private and Public Keys to be able to sign Files AGAIN  !!!!! -- #
# -------------------------------------------------------------------------------------------------#
export EPHEMERAL_KEYRING_FOLDER_ZERO=$(mktemp -d)
export RESTORE_GPG_TMP_DIR=$(mktemp -d)
export RESTORED_GPG_PUB_KEY_FILE="$(pwd)/graviteebot.gpg.pub.key"
export RESTORED_GPG_PRIVATE_KEY_FILE="$(pwd)/graviteebot.gpg.priv.key"

chmod 700 ${EPHEMERAL_KEYRING_FOLDER_ZERO}
export GNUPGHOME=${EPHEMERAL_KEYRING_FOLDER_ZERO}
# gpg --list-secret-keys
# gpg --list-pub-keys
gpg --list-keys

secrethub read --out-file ${RESTORED_GPG_PUB_KEY_FILE} "${SECRETHUB_ORG}/${SECRETHUB_REPO}/graviteebot/gpg/pub_key"
secrethub read --out-file ${RESTORED_GPG_PRIVATE_KEY_FILE} "${SECRETHUB_ORG}/${SECRETHUB_REPO}/graviteebot/gpg/private_key"

gpg --import ${RESTORED_GPG_PRIVATE_KEY_FILE}
gpg --import ${RESTORED_GPG_PRIVATE_KEY_FILE}

# now we trust ultimately the Public Key in the Ephemeral Context,
export GRAVITEEBOT_GPG_SIGNING_KEY_ID=$(secrethub read "${SECRETHUB_ORG}/${SECRETHUB_REPO}/graviteebot/gpg/key_id")
echo "GRAVITEEBOT_GPG_SIGNING_KEY_ID=[${GRAVITEEBOT_GPG_SIGNING_KEY_ID}]"

echo -e "5\ny\n" |  gpg --command-fd 0 --expert --edit-key ${GRAVITEEBOT_GPG_SIGNING_KEY_ID} trust

# ------------------------------------------------------------------------------------------------ #
# ------------------------------------------------------------------------------------------------ #
# ------------------------------------------------------------------------------------------------ #
# ------------------------------------------------------------------------------------------------ #
# ------------------------------------------------------------------------------------------------ #
# ------------------------------------------------------------------------------------------------ #
# -- TESTS --   Testing using the Restored GPG Key :                                   -- TESTS -- #
# -- TESTS --   to sign a file, and verify file signature                              -- TESTS -- #
# ------------------------------------------------------------------------------------------------ #
# ------------------------------------------------------------------------------------------------ #
# ------------------------------------------------------------------------------------------------ #
# ------------------------------------------------------------------------------------------------ #
# ------------------------------------------------------------------------------------------------ #
# ------------------------------------------------------------------------------------------------ #

# ++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++ #
# ------------------------------------------------------------------------------------------------ #
# -- TESTS --                          First Let's Sign a file                         -- TESTS -- #
# ------------------------------------------------------------------------------------------------ #
cat >./some-file-to-sign.txt <<EOF
Hey I ma sooo important a file that
I am in a file which is going to be signed to proove my integrity
EOF

export GRAVITEEBOT_GPG_PASSPHRASE=$(secrethub read "${SECRETHUB_ORG}/${SECRETHUB_REPO}/graviteebot/gpg/passphrase")



# echo "${GRAVITEEBOT_GPG_PASSPHRASE}" | gpg --pinentry-mode loopback --passphrase-fd 0 --sign ./some-file-to-sign.txt

# ---
# That's Jean-Baptiste Lasselle's GPG SIGNING KEY ID for signing git commits n tags (used as example)
export GPG_SIGNING_KEY_ID=7B19A8E1574C2883
# ---
# That's the GPG_SIGNING_KEY used buy the "Gravitee.io Lab Bot" for git and signing any file
export GRAVITEEBOT_GPG_SIGNING_KEY_ID=$(gpg --list-signatures -a "${GRAVITEEBOT_GPG_USER_NAME} (${GRAVITEEBOT_GPG_USER_NAME_COMMENT}) <${GRAVITEEBOT_GPG_USER_EMAIL}>" | grep 'sig' | tail -n 1 | awk '{print $2}')
echo "GRAVITEEBOT_GPG_SIGNING_KEY_ID=[${GRAVITEEBOT_GPG_SIGNING_KEY_ID}]"

gpg --keyid-format LONG -k "0x${GRAVITEEBOT_GPG_SIGNING_KEY}"

echo "${GRAVITEEBOT_GPG_PASSPHRASE}" | gpg -u "0x${GRAVITEEBOT_GPG_SIGNING_KEY}" --pinentry-mode loopback --passphrase-fd 0 --sign ./some-file-to-sign.txt
echo "${GRAVITEEBOT_GPG_PASSPHRASE}" | gpg -u "0x${GRAVITEEBOT_GPG_SIGNING_KEY}" --pinentry-mode loopback --passphrase-fd 0 --detach-sign ./some-file-to-sign.txt



# -- #
# Will be interactive for private key : you
# will have to type your GPG password
# gpg --export-secret-key -a "${GRAVITEEBOT_GPG_USER_NAME} <${GRAVITEEBOT_GPG_USER_EMAIL}>" | tee ${GPG_PRIVATE_KEY_FILE}

# ------------------------------------------------------------------------------------------------ #
# # - To sign a GPG Key  with 1 specific private keys
# gpg --local-user 0xDEADBEE5 --sign file
# # - To sign a GPG Key  with 2 private keys
# gpg --local-user 0xDEADBEE5 --local-user 0x12345678 --sign file
# # - To sign a GPG Key  with 1 specific private keys
# gpg -u 0xDEADBEE5 --sign file
# # - To sign a GPG Key  with 2 private keys
# gpg -u 0xDEADBEE5 --local-user 0x12345678 --sign file
# ------------------------------------------------------------------------------------------------ #
echo "# ------------------------------------------------------------------------------------------------ #"
echo "the [$(pwd)/some-file-to-sign.txt] file is the file which was signed"
ls -allh ./some-file-to-sign.txt
echo "the [$(pwd)/some-file-to-sign.txt.sig] file is the signed file which was signed, and has its signature embedded"
ls -allh ./some-file-to-sign.txt.gpg
echo "the [$(pwd)/some-file-to-sign.txt.sig] file is the (detached) signature of the file which was signed"
ls -allh ./some-file-to-sign.txt.sig
echo "# ------------------------------------------------------------------------------------------------ #"
echo "In software, we use detached signatures, because when you sign a very "
echo "big size file, distributing the signature does not force distributing a very big file"
echo "# ------------------------------------------------------------------------------------------------ #"


# ++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++ #
# ------------------------------------------------------------------------------------------------ #
# -- TESTS --   Now test verifying the signed file, using its detached signature       -- TESTS -- #
# ------------------------------------------------------------------------------------------------ #

echo "  Now testing verifying the file with its detached signature :"
gpg --verify ./some-file-to-sign.txt.sig some-file-to-sign.txt
echo "# ------------------------------------------------------------------------------------------------ #"
echo "  Now testing verifying the file with its detached signature, in another Ephemeral GPG Keyring "
echo "# ------------------------------------------------------------------------------------------------ #"
export EPHEMERAL_KEYRING_FOLDER_TWO=$(mktemp -d)
chmod 700 ${EPHEMERAL_KEYRING_FOLDER_TWO}
export GNUPGHOME=${EPHEMERAL_KEYRING_FOLDER_TWO}
# gpg --list-secret-keys
# gpg --list-pub-keys
gpg --list-keys
echo "# ------------------------------------------------------------------------------------------------ #"
unset GNUPGHOME
echo "  First, without resetting GNUPGHOME env. var.  "
echo "  (we are still in the default Keyring for the current Linux User, so verifying will be successful) "
echo "# ------------------------------------------------------------------------------------------------ #"
gpg --verify ./some-file-to-sign.txt.sig some-file-to-sign.txt
echo "# ------------------------------------------------------------------------------------------------ #"
echo "  Now let's switch to the created Ephemeral GPG Keyring (Ephemeral GPG Context)"
echo "# ------------------------------------------------------------------------------------------------ #"
export GNUPGHOME=${EPHEMERAL_KEYRING_FOLDER_TWO}
# gpg --list-secret-keys
# gpg --list-pub-keys
gpg --list-keys
echo "# ------------------------------------------------------------------------------------------------ #"
echo "  Ok, there is no GPG Public key in this Ephemral GPG context"
echo "  That's why verifying the signed file with its detached signature, will fail : "
echo "    => a GPG signature is \"bound\" to its associated Public Key "
echo "    => GPG signature is Asymetric Cryptography (very important)"
echo "# ------------------------------------------------------------------------------------------------ #"
gpg --verify ./some-file-to-sign.txt.sig some-file-to-sign.txt

# now we import the Public Key in the Ephemeral Context, trust it ultimately, and verify the file signature again
gpg --import "${GPG_PUB_KEY_FILE}"
# now we trust ultimately the Public Key in the Ephemeral Context,
export GRAVITEEBOT_GPG_SIGNING_KEY_ID=$(gpg --list-signatures -a "${GRAVITEEBOT_GPG_USER_NAME} (${GRAVITEEBOT_GPG_USER_NAME_COMMENT}) <${GRAVITEEBOT_GPG_USER_EMAIL}>" | grep 'sig' | tail -n 1 | awk '{print $2}')
echo "GRAVITEEBOT_GPG_SIGNING_KEY_ID=[${GRAVITEEBOT_GPG_SIGNING_KEY_ID}]"

echo -e "5\ny\n" |  gpg --command-fd 0 --expert --edit-key ${GPG_SIGNING_KEY_ID} trust

# ++
# ++ To ultimately trust ALL Keys :
# for fpr in $(gpg --list-keys --with-colons  | awk -F: '/fpr:/ {print $10}' | sort -u); do  echo -e "5\ny\n" |  gpg --command-fd 0 --expert --edit-key $fpr trust; done
# ++

# ++
# And finally verify the file signature again
gpg --verify ./some-file-to-sign.txt.sig some-file-to-sign.txt


```
