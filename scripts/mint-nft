require("dotenv").config();

const express = require("express");

const date = require("date-and-time");

const app = new express();

const { createAlchemyWeb3 } = require("@alch/alchemy-web3");

const bodyParser = require("body-parser");

const pinataSDK = require("@pinata/sdk");

const pinata = pinataSDK(
  "821a1b06d467163f6381",
  "6d43ee28cd6d610e6b898757089fe898fc243e5e1d308edae653f010dd01a133"
);

const PUBLIC_KEY = process.env.PUBLIC_KEY;

const PRIVATE_KEY = process.env.PRIVATE_KEY;

const API_URL = process.env.API_URL;

const web3 = createAlchemyWeb3(API_URL);

const now = new Date();

const contract = require("../artifacts/contracts/MyNFT.sol/MyNFT.json");

const contractAddress = "0x09B456cfe9E7a9F04A409B982DCb2A7AbE713B39";

const nftContract = new web3.eth.Contract(contract.abi, contractAddress);

app.use(bodyParser.urlencoded({ extended: false }));

app.use(bodyParser.json());

//create transaction

//API for Minting Url

app.post("/mintnft", async (req, res) => {
  try {
    let rsp = {};

    const data = {
      name: req.body.name,
      color: req.body.color,
      description: req.body.description,
      tokenURI: req.body.tokenURI,
      price: req.body.price,
      dateTime: req.body.dateTime,
      to: req.body.to,
    };

    pinata.pinJSONToIPFS(data).then(async (result) => {
      let hsh = "https://gateway.pinata.cloud/ipfs/" + result.IpfsHash;
      const nonce = await web3.eth.getTransactionCount(PUBLIC_KEY, "latest"); //get latest nonce

      const pattern = date.compile("ddd, MMM DD YYYY");

      const datetime = date.format(now, pattern); //the transaction

      const tx = {
        from: PUBLIC_KEY,

        to: contractAddress,
        nonce: nonce,

        gas: 500000, //data: nftContract.methods.mintNFT(PUBLIC_KEY,hsh,"red","5500","rollex_watch").encodeABI(),

        data: nftContract.methods
          .mintNFT(
            data.name,

            data.color,

            data.description,

            data.price,

            data.dateTime,

            data.to,

            PUBLIC_KEY,

            data.tokenURI
          )
          .encodeABI(),
      };

      const signPromise = web3.eth.accounts.signTransaction(tx, PRIVATE_KEY);

      signPromise

        .then((signedTx) => {
          web3.eth.sendSignedTransaction(
            signedTx.rawTransaction,

            async function (err, hash) {
              if (!err) {
                console.log("The hash of your transaction is: ", hash);

                const txDetails = await web3.eth.getTransaction(hash);

                rsp["TxHash"] = hash;

                rsp["Ipfs Url"] = hsh;

                rsp["MetaData"] = data;

                rsp["Contract Address"] = txDetails.to;

                res.json(rsp);

                res.end();
              } else {
                console.log(
                  "Something went wrong when submitting your transaction:",
                  err
                );

                return err;
              }
            }
          );
        })

        .catch((err) => {
          console.log(" Promise failed:", err);
        });
    });
  } catch (error) {
    res.write(404).send(error);
  }
});

//API for TransferNFT Url

app.post("/transfernft", async (req, res) => {
  try {
    let rsp = {};

    const data = {
      to: req.body.to,

      tokenid: req.body.tokenid,
    };

    console.log(data); //create transaction

    const nonce = await web3.eth.getTransactionCount(PUBLIC_KEY, "latest"); //get latest nonce //the transaction

    const tx = {
      from: PUBLIC_KEY,

      to: contractAddress,

      nonce: nonce,

      gas: 500000,

      data: nftContract.methods

        .transferFrom(PUBLIC_KEY, data.to, data.tokenid)

        .encodeABI(),
    }; //sign Transcation

    const signPromise = web3.eth.accounts.signTransaction(tx, PRIVATE_KEY);

    signPromise

      .then((signedTx) => {
        web3.eth.sendSignedTransaction(
          signedTx.rawTransaction,

          async function (err, hash) {
            if (!err) {
              console.log("The hash of your transaction is: ", hash);

              const txDetails = await web3.eth.getTransaction(hash);

              rsp["TxHash"] = hash;

              rsp["Input-Data"] = data;

              rsp["Contract Address"] = txDetails.to;

              res.json(rsp);

              res.end();
            } else {
              console.log(
                "Something went wrong when submitting your transaction:",

                err
              );

              return err;
            }
          }
        );
      })

      .catch((err) => {
        console.log(" Promise failed:", err);
      });
  } catch (err) {
    res.status(400).send(err);
  }
});

//serving on this port

app.listen(5000, () => {
  console.log("Lisitng on Port 5000");
});
