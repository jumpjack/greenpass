 OVERVIEW
===========

 Pure javascript greenpass QR code decoder in browser
 
 - Main page: <a href="http://jumpjack.altervista.org/greenpass/">link</a>
 - Test page for developers (only console output): <a href="https://jumpjack.altervista.org/greenpass/mini.html">link</a>
 - Qrcode library: https://github.com/klonikar/qrcodejs
 - Test data taken  from: https://github.com/eu-digital-green-certificates/dgc-testdata/tree/main/IT

--------------------

With this tool you can decode, view and understand all the data actually contained in your greenpass/QR, which are instead not visible using the standard app to check green pass validity, but you can't validate (i.e. verify if QR is valid and legal), this function is not implemented; but the script exposes variables **headers1, headers2, cbor_data,** and  **signature**; greenpass JSON data are in **cbor_data**, the other variables are for validation.

You don't need to upload your QR code to any site, all processing is performed offline: just drag the QR code on the specified page, or paste the string resulting from QR code reading by any app, then click a button and see all your data.

The decoding process is:

QR code --> **QR DECODER** --> RAW QR-decoded string --> **BASE45 decoder** --> zlib compressed string --> **pako library** --> COSE object --> **CBOR decoder** --> CBOR object --**> CBOR decoder** --> final JSON file

Technical details
=================

The line which provides as result the raw COSE object is:

`[headers1, headers2, cbor_data, signature] = CBOR.decode(unzipped);`

It can also be found in other sources as:

`[protected_header, unprotected_header, cbor_data, signatures] = CBOR.decode(unzipped);`

The 4 elements of the CBOR array have these formats:

 - [0] = protected_header = CBORType.ByteString  ---> To be further decodec by CBOR.decode()
 - [1] = unprotected_header = CBORType.Map         ---> Empty
 - [2] = cbor_data = CBORType.ByteString  ---> To be further decodec by CBOR.decode()
 - [3] = signatures CBORType.ByteString  ---> Raw TBW

Example data (COSE sequence (i.e. CBOR signed sequence) derived from unzipping of QR code / base45 data):

	d2844da2044839301768cdda05130126a0590101a4041a6194e898061a60a78c8801624954390103a101a4617681aa62646e02626d616d4f52472d3130303033303231356276706a313131393334393030376264746a323032312d30342d313062636f62495462636978263031495445373330304531414232413834433731393030344631303344434231463730412336626d706c45552f312f32302f313532386269736249546273640262746769383430353339303036636e616da463666e746944493c43415052494f62666e6944692043617072696f63676e746d4d4152494c553c54455245534162676e6e4d6172696cc3b9205465726573616376657265312e302e3063646f626a313937372d30362d31365840a4ee9016c1a74ccf9caab905492d698f6992a8fa30c20db6180f06040c4870a845bb4b3a1ce3f4ed529cc78e66322547d62637c74ab17919c0aa52a614795e9e


In raw format, the COSE byte sequence of the greenpass is:

	  d2                -- Tag #18 - Number identifying the used algorithm for signature in cose sign.js? (18dec, 0x12 = "Sign1Tag" algorithm)
	    84              -- Array, 4 items
	      4d            -- Bytes, length: 13
		            [snip] - Protected array, containing Algorithm and KID
	      a0            -- [1], {} - Unprotected array, empty
	      59            -- Bytes, length next 2 bytes
		0101        -- Bytes, length: 257
			    [snip] - The JSON structure contanining actual greenpas data
	      58            -- Bytes, length next 1 byte
		40          -- Bytes, length: 64
			   [snip] - The signature of the CBOR object (format unknown)

Reference for signature algorithm: [cose "sign.js" file](https://github.com/erdtman/cose-js/blob/004751a1bd88265cce768f3466ea86ecbf2945fa/lib/sign.js#L230)
 
 
Both protected and unprotected header have this structure:


	Generic_Headers = (
	       ? 1 => int / tstr,  ; algorithm identifier           IN GREENPASS
	       ? 2 => [+label],    ; criticality
	       ? 3 => tstr / int,  ; content type
	       ? 4 => bstr,        ; key identifier                 IN GREENPASS
	       ? 5 => bstr,        ; IV
	       ? 6 => bstr,        ; Partial IV
	       ? 7 => COSE_Signature / [+COSE_Signature] ; Counter signature
	   )

 Other view (source: https://datatracker.ietf.org/doc/html/rfc8152#page-15 , ):
 
 
	   +-----------+-------+----------------+-------------+----------------+
	   | Name      | Label | Value Type     | Value       | Description    |
	   |           |       |                | Registry    |                |
	   +-----------+-------+----------------+-------------+----------------+
	   | alg       | 1     | int / tstr     | COSE        | Cryptographic  |           IN GREENPASS
	   |           |       |                | Algorithms  | algorithm to   |
	   |           |       |                | registry    | use            |
	   | crit      | 2     | [+ label]      | COSE Header | Critical       |
	   |           |       |                | Parameters  | headers to be  |
	   |           |       |                | registry    | understood     |
	   | content   | 3     | tstr / uint    | CoAP        | Content type   |
	   | type      |       |                | Content-    | of the payload |
	   |           |       |                | Formats or  |                |
	   |           |       |                | Media Types |                |
	   |           |       |                | registries  |                |
	   | kid       | 4     | bstr           |             | Key identifier |           IN GREENPASS
	   | IV        | 5     | bstr           |             | Full           |
	   |           |       |                |             | Initialization |
	   |           |       |                |             | Vector         |
	   | Partial   | 6     | bstr           |             | Partial        |
	   | IV        |       |                |             | Initialization |
	   |           |       |                |             | Vector         |
	   | counter   | 7     | COSE_Signature |             | CBOR-encoded   |
	   | signature |       | / [+           |             | signature      |
	   |           |       | COSE_Signature |             | structure      |
	   |           |       | ]              |             |                |
	   +-----------+-------+----------------+-------------+----------------+

   
 In greenpass only field "1" and field "4" are present: **Algorithm identifie**r and **KEY identifier** (KID)
 
 Example of decoded protected header:
 
	{
		4: h'39301768CDDA0513',   
		1: -7
	}

Field named "1" represents the used algorithm as per following [table](https://datatracker.ietf.org/doc/html/rfc8152#page-39):
 
		      +-------+-------+---------+------------------+
		      | Name  | Value | Hash    | Description      |
		      +-------+-------+---------+------------------+
		      | ES256 | -7    | SHA-256 | ECDSA w/ SHA-256 |
		      | ES384 | -35   | SHA-384 | ECDSA w/ SHA-384 |
		      | ES512 | -36   | SHA-512 | ECDSA w/ SHA-512 |
		      +-------+-------+---------+------------------+

			      Table 5: ECDSA Algorithm Values

Field named "4" contains the Key Identifier (KID), and index to identify the Signer Certificate to be used to check validity of signature.
The hexadecimal sequence must be converted to BASE64; the provided example  "39301768CDDA0513" [converts to](https://base64.guru/converter/encode/hex) "OTAXaM3aBRM=", which is the KID for Italian certificate:

	  "IT" : {
	      "eku" : { },
	      "keys" : [ {
		"kty" : "EC",
		"x5t#S256" : "OTAXaM3aBRO0X4O8aRMXfVP9MIk97f9js1JmAQvwkWo",
		"crv" : "P-256",
		"kid" : "OTAXaM3aBRM=",
		"x5c" : [ "MIIEHjCCAgagAwIBAgIUM5lJeGCHoRF1raR6cbZqDV4vPA8wDQYJKoZIhvcNAQELBQAwTjELMAkGA1UEBhMCSVQxHzAdBgNVBAoMFk1pbmlzdGVybyBkZWxsYSBTYWx1dGUxHjAcBgNVBAMMFUl0YWx5IERHQyBDU0NBIFRFU1QgMTAeFw0yMTA1MDcxNzAyMTZaFw0yMzA1MDgxNzAyMTZaME0xCzAJBgNVBAYTAklUMR8wHQYDVQQKDBZNaW5pc3Rlcm8gZGVsbGEgU2FsdXRlMR0wGwYDVQQDDBRJdGFseSBER0MgRFNDIFRFU1QgMTBZMBMGByqGSM49AgEGCCqGSM49AwEHA0IABDSp7t86JxAmjZFobmmu0wkii53snRuwqVWe3/g/wVz9i306XA5iXpHkRPZVUkSZmYhutMDrheg6sfwMRdql3aajgb8wgbwwHwYDVR0jBBgwFoAUS2iy4oMAoxUY87nZRidUqYg9yyMwagYDVR0fBGMwYTBfoF2gW4ZZbGRhcDovL2NhZHMuZGdjLmdvdi5pdC9DTj1JdGFseSUyMERHQyUyMENTQ0ElMjBURVNUJTIwMSxPPU1pbmlzdGVybyUyMGRlbGxhJTIwU2FsdXRlLEM9SVQwHQYDVR0OBBYEFNSEwjzu61pAMqliNhS9vzGJFqFFMA4GA1UdDwEB/wQEAwIHgDANBgkqhkiG9w0BAQsFAAOCAgEAIF74yHgzCGdor5MaqYSvkS5aog5+7u52TGggiPl78QAmIpjPO5qcYpJZVf6AoL4MpveEI/iuCUVQxBzYqlLACjSbZEbtTBPSzuhfvsf9T3MUq5cu10lkHKbFgApUDjrMUnG9SMqmQU2Cv5S4t94ec2iLmokXmhYP/JojRXt1ZMZlsw/8/lRJ8vqPUorJ/fMvOLWDE/fDxNhh3uK5UHBhRXCT8MBep4cgt9cuT9O4w1JcejSr5nsEfeo8u9Pb/h6MnmxpBSq3JbnjONVK5ak7iwCkLr5PMk09ncqG+/8Kq+qTjNC76IetS9ST6bWzTZILX4BD1BL8bHsFGgIeeCO0GqalFZAsbapnaB+36HVUZVDYOoA+VraIWECNxXViikZdjQONaeWDVhCxZ/vBl1/KLAdX3OPxRwl/jHLnaSXeqr/zYf9a8UqFrpadT0tQff/q3yH5hJRJM0P6Yp5CPIEArJRW6ovDBbp3DVF2GyAI1lFA2Trs798NN6qf7SkuySz5HSzm53g6JsLY/HLzdwJPYLObD7U+x37n+DDi4Wa6vM5xdC7FZ5IyWXuT1oAa9yM4h6nW3UvC+wNUusW6adqqtdd4F1gHPjCf5lpW5Ye1bdLUmO7TGlePmbOkzEB08Mlc6atl/vkx/crfl4dq1LZivLgPBwDzE8arIk0f2vCx1+4=" ],
		"x" : "NKnu3zonECaNkWhuaa7TCSKLneydG7CpVZ7f-D_BXP0",
		"y" : "i306XA5iXpHkRPZVUkSZmYhutMDrheg6sfwMRdql3aY"
	      } ]
	    },

The "x5c" is the certificate, to be enclosed between "-----BEGIN CERTIFICATE-----" and "-----END CERTIFICATE-----" strings.

The signature
-------------

Last element of CBOR array contains the signature of the greenpass; it's made of 64 bytes, which must be split in 2 to get values "R" and "S"; in our example:

- A4EE9016C1A74CCF9CAAB905492D698F6992A8FA30C20DB6180F06040C4870A845BB4B3A1CE3F4ED529CC78E66322547D62637C74AB17919C0AA52A614795E9E 

This is a sequence of alphamumeric couples, each one represeting an hexadecimal number. It should (to be confirmed) represent a signature in P1363 format, which should be converted to DER format to be used in OPENSSL.

Splitted:

 -  R: A4EE9016C1A74CCF9CAAB905492D698F6992A8FA30C20DB6180F06040C4870A8
  - S: 45BB4B3A1CE3F4ED529CC78E66322547D62637C74AB17919C0AA52A614795E9E 

This article explains how to use these numbers to perform a local signature verification using openssl:

http://www.corentindupont.info/blog/posts/Programming/2021-08-13-GreenPass.html

ANS.1 DER checker: https://lapo.it/asn1js/

According to [RFC8152](https://datatracker.ietf.org/doc/html/rfc8152#section-4.4), signature verification procedure is:


>   1.  Create a Sig_structure object and populate it with the
>       appropriate fields.
>
>   2.  Create the value ToBeSigned by encoding the Sig_structure to a
>       byte string, using the encoding described in Section 14.
>
>   3.  Call the signature verification algorithm passing in K (the key
>       to verify with), alg (the algorithm used sign with), ToBeSigned
>       (the value to sign), and sig (the signature to be verified).

 - **K**: verifier
 - **alg**:  -7/ES256/SHA-256
 - **sig**: raw greenpass signature hex string representing 64 bytes	
 - **ToBeSigned**: cbor encoding of:


```const SigStructure = [
'Signature1', // text string
p, //    cbor.encode(cbor.decodeFirstSync(protected_header)); /// Decode and re-encode?!? Hence unchanged?
externalAAD, // Always EMPTY_BUFFER in greenpass?
plaintext // cbor_data from  [headers1, headers2, cbor_data, signature] = CBOR.decode(unzipped);
];
```

Hence:

	const SigStructure = [
		'Signature1',
		protected_header,
		EMPTY_BUFFER,
		cbor_data
	];

Types:

	   Sig_structure = [
	       context : "Signature1",
	       body_protected : empty_or_serialized_map,
	       ? sign_protected : empty_or_serialized_map,
	       external_aad : bstr,
	       payload : bstr
	
This Sig_structure can be seen in [SIGN.JS file of COSE](https://github.com/erdtman/cose-js/blob/004751a1bd88265cce768f3466ea86ecbf2945fa/lib/sign.js#L285) javascript library.
	
	       
Program flow for checking signature:

	unzipped = zlib.inflateSync(base45Data); // raw uncompressed greenpass = COSE object (CBOR signed) = [p, u, payload, signature] ([here](https://github.com/ministero-salute/dcc-utils/blob/99e255d46997b9324748733d98ab151fcb4a3162/src/dcc.js#L12))
	COSEraw = unzipped;
	cose.sign.verify(
				COSEraw, 
				{
					key: {
					// x, y and KID are extraxted from public certificate taken from 'https://raw.githubusercontent.com/bcsongor/covid-pass-verifier/35336fd3c0ff969b5b4784d7763c64ead6305615/src/data/certificates.json'
					x: Buffer.from(signature.pub.x), 
					y: Buffer.from(signature.pub.y),
					kid: Buffer.from(signature.kid),
					},
				}
			); // (COSE library [sign.js](https://github.com/erdtman/cose-js/blob/004751a1bd88265cce768f3466ea86ecbf2945fa/lib/sign.js))
		
		COSE = CBOR.decode(COSEraw);

		verifyInternal({
			key: {
				x: Buffer.from(signature.pub.x), // x, y and KID are extraxted from public certificate
				y: Buffer.from(signature.pub.y),
				kid: Buffer.from(signature.kid),
				},
			}, {}, COSE); //in [sign.js](https://github.com/erdtman/cose-js/blob/004751a1bd88265cce768f3466ea86ecbf2945fa/lib/sign.js#L220)

			doVerify(     
				SigStructure = 
					[
						'Signature1',
						COSE[0],
						EMPTY_BUFFER,
						COSE[2]
					],
				verifier = 
				{
					key: 
						{
							x: Buffer.from(signature.pub.x), // x, y and KID are extraxted from public certificate
							y: Buffer.from(signature.pub.y),
							kid: Buffer.from(signature.kid),
						},
				}, 
				alg, // Always -7 for greenpass?
				sig  // Raw signature of greenpass (64 bytes as hex string)
				);
				
				AlgFromTags[-7] = { 'sign': 'ES256', 'digest': 'SHA-256' };
				COSEAlgToNodeAlg = {  'ES256': { 'sign': 'p256', 'digest': 'sha256' }, // .... }; /Only ES256 used in greenpass?
				nodeAlg = COSEAlgToNodeAlg[AlgFromTags[alg].sign] // = COSEAlgToNodeAlg[AlgFromTags[-7].sign] = COSEAlgToNodeAlg[ES256] = { 'sign': 'p256', 'digest': 'sha256' }
				const hash = crypto.createHash(nodeAlg.digest); // =  crypto.createHash('sha256')    (CRYPTO library)
				hash.update(cbor.encode(SigStructure));
				const msgHash = hash.digest();
				const pub = { 'x': Buffer.from(signature.pub.x), 'y': Buffer.from(signature.pub.y) };  // Data from public certificate
				const ec = new EC(nodeAlg.sign); // = new EC('p256')  (Elliptic curve library)
				const key = ec.keyFromPublic(pub); 
				sig = { 'r': sig.slice(0, sig.length / 2), 's': sig.slice(sig.length / 2) }; // Signature is split in 2 halves: R and S
				key.verify(msgHash, sig); // Final call to signature check algorithm in Elliptic Curve library

			  
			  


CBOR online decoders
====================

Following sites can be used to decode online CBOR bytes sequence without writing lines of code:
 - http://cbor.me/
 - https://hildjj.github.io/node-cbor/example/index-bf.html


-------------

A full and official description of greenpass data (i.e. JSON fields) is available in "<a href="https://ec.europa.eu/health/sites/default/files/ehealth/docs/covid-certificate_json_specification_en.pdf">Guidelines on Technical Specifications for EU Digital COVID Certificates - JSON Schema Specification</a> v. 1.3.0" from "<a href="https://ec.europa.eu/health/">eHealth Network</a>" (old version 1.0 is <a href="https://ec.europa.eu/health/sites/default/files/ehealth/docs/digital-green-certificates_dt-specifications_en.pdf">here</a>; changelog is unknown).

Here you can find some test QR codes from official source, in case you want to test the tol before using it: <a href="https://github.com/eu-digital-green-certificates/dgc-testdata/tree/main/IT/png">link</a>


**QR code for a tested person**

<img src="https://github.com/jumpjack/greenpass/raw/main/demo3-tested.png">

    HC1:6BFOXN%TS3DH0YOJ58S S-W5HDC *M0IIE 1C9B5G2+$NP-OP-IA%N%QHRJPC%OQHIZC4.OI:OIG/Q80P2W4VZ0K1H$$0CNN62PK.G +AG5T01HJCAMKNAB5S.8%*8Z95%9EMP8N22MM42WFCD9C2AKIJKIJM1MQIAY.D-7A4KE0PLV1ARKF.GH5$C4-9GGIUEC0QE1JAF.714NTPINRQ3.VR+P0$J2*N$*SB-G9+RT*QFNI2X02%KYZPQV6YP8412HOA-I0+M9GPEGPEMH0SJ4OM9*1B+M96K1HK2YJ2PI0P:65:41ZSW$P*CM-NT0 2$88L/II 05B9.Z8T*8Y1VM:KCY07LPMIH-O9XZQ4H9IZBP%D2U3+KGP2W2UQNG6-E6+WJTK1%J6/UI2YUELE+W35T7+H8NH8DRG+PG.UIZ$U%UF*QHOOENBU621TW5XW5HS9+I010H% 0R%0ZD5CC9T0HP8TCNNI:CQ:G172DX8FZV3U9W-HNPPQ N2KV 2VHDHO:2XAV:FB+18DRR%%VQ F60LF6K 38GK8LGG4U7UP6*S4QBR-F97FRONPKZS+P9$5W1CAV37KD48ERCRH
    
    

**QR code for a healed person**

<img src="https://github.com/jumpjack/greenpass/raw/main/demo2-healed.png">

    HC1:6BFOXN%TS3DH0YOJ58S S-W5HDC *MEB2B2JJ59J-9BC6:X9NECX0AKQC:3DCV4*XUA2P-FHT-H4SI/J9WVHWVH+ZEOV1J$HNTICZUBOM*LP$V25$0Q:J40IA3L/*84-5%:C92JN*4CY0*%9F/8J2P4.818T+:IX3M3.96RPVD9J-OZT1-NT0 2$$0$2PZX69B9VCDHI2/T9TU1BPIJKH/T7B-S-*O/Y41FD+X49+5Z-6%.HDD8R6W1FDJGJSFJ/4Q:T0.KJTNP8EFULNC:HA0K5HKRB4TD85LOLF92GF.3O.Z8CC7-2FQYG$%21 2O*4R60NM8JI0EUGP$I/XK$M8ZQE6YB9M66P8N31I.ROSK%IA1Q2N53Q-OQ2VC6E26T11ROSNK5W-*H+MJ%0RGZVGWNURI75RBSQSHLH1JG*CMH2.-S$7VX6N*Z1881J7G.F9I+SV06F+1M*93%D



**QR code for a vaccinated person:**

<img src="https://github.com/jumpjack/greenpass/raw/main/demo1-vaccinated.png">

    HC1:6BFOXN%TS3DH0YOJ58S S-W5HDC *M0II5XHC9B5G2+$N IOP-IA%NFQGRJPC%OQHIZC4.OI1RM8ZA.A5:S9MKN4NN3F85QNCY0O%0VZ001HOC9JU0D0HT0HB2PL/IB*09B9LW4T*8+DCMH0LDK2%K:XFE70*LP$V25$0Q:J:4MO1P0%0L0HD+9E/HY+4J6TH48S%4K.GJ2PT3QY:GQ3TE2I+-CPHN6D7LLK*2HG%89UV-0LZ 2ZJJ524-LH/CJTK96L6SR9MU9DHGZ%P WUQRENS431T1XCNCF+47AY0-IFO0500TGPN8F5G.41Q2E4T8ALW.INSV$ 07UV5SR+BNQHNML7 /KD3TU 4V*CAT3ZGLQMI/XI%ZJNSBBXK2:UG%UJMI:TU+MMPZ5$/PMX19UE:-PSR3/$NU44CBE6DQ3D7B0FBOFX0DV2DGMB$YPF62I$60/F$Z2I6IFX21XNI-LM%3/DF/U6Z9FEOJVRLVW6K$UG+BKK57:1+D10%4K83F+1VWD1NE

**Generic JSON file for greenpass:**

    "JSON": {
        "ver": <version information>,
        "nam": {
        <person name information>
    },
    "dob": <date of birth>,
    "v" or "t" or "r": [
        {<vaccination dose or test or recovery information, one entry>}
        ]
    }
    
 ---------
 
**JSON file for a tested person:**

    {
        "1": "IT",
        "4": 1637148826,
        "6": 1621593226,
        "-260": {
                "1": {
                    "t": [
                            {
                                "sc": "2021-05-03T10:27:15Z",
                                "ma": "1232",
                                "dr": "2021-05-11T12:27:15Z",
                                "tt": "LP6464-4",
                                "nm": "Roche LightCycler qPCR",
                                "co": "IT",
                                "tc": "Policlinico Umberto I",
                                "ci": "01IT053059F7676042D9BEE9F874C4901F9B#3",
                                "is": "IT",
                                "tg": "840539006",
                                "tr": "260415000"
                            }
		                      	],
			                 "nam": {
			                             "fnt": "DI<CAPRIO",
			                             "fn": "Di Caprio",
			                             "gnt": "MARILU<TERESA",
			                             "gn": "Marilù Teresa"
			                        },
			                 "ver": "1.0.0",
			                 "dob": "1977-06-16"
			                 }
            }
    }


**JSON file for healed person:**

    {
            "1": "IT",
            "4": 1637148823,
            "6": 1621593223,
            "-260": {
                "1": {
                    "r": [
                            {
                                "du": "2021-10-31",
                                "co": "IT",
                                "ci": "01ITA65E2BD36C9E4900B0273D2E7C92EEB9#1",
                                "is": "IT",
                                "tg": "840539006",
                                "df": "2021-05-04",
                                "fr": "2021-05-02"
                            }
                         ],
			                 "nam": {
			                         "fnt": "DI<CAPRIO",
			                         "fn": "Di Caprio",
			                         "gnt": "MARILU<TERESA",
			                         "gn": "Marilù Teresa"
			                        },
			                 "ver": "1.0.0",
			                 "dob": "1977-06-16"
	               	}
            }
    }


**JSON file for vaccinated person:**

    {
        "1": "IT",
        "4": 1637148824,
        "6": 1621593224,
        "-260": {
            "1": {
                "v": [
                    {
                        "dn": 2,
                        "ma": "ORG-100030215",
                        "vp": "1119349007",
                        "dt": "2021-04-10",
                        "co": "IT",
                        "ci": "01ITE7300E1AB2A84C719004F103DCB1F70A#6",
                        "mp": "EU/1/20/1528",
                        "is": "IT",
                        "sd": 2,
                        "tg": "840539006"
                    }
                ],
                "nam": {
                    "fnt": "DI<CAPRIO",
                    "fn": "Di Caprio",
                    "gnt": "MARILU<TERESA",
                    "gn": "Marilù Teresa"
                },
                "ver": "1.0.0",
                "dob": "1977-06-16"
            }
        }
    }

------------

Basic javascript steps (see source for details):

    source = document.getElementById("encoded").value; // Raw QR reader output
    decoded = decode(source).enc; // decode BASE45
    COSEbin =  pako.inflate(decode(source).raw); // decompress
    
    // convert for CBOR library:
    COSE = buf2hex(COSEbin); 
    var typedArray = new Uint8Array(COSE.match(/[\da-f]{2}/gi).map(function (h) {
          return parseInt(h, 16)
        })) // https://stackoverflow.com/questions/43131242/how-to-convert-a-hexadecimal-string-of-data-to-an-arraybuffer-in-javascript
    var unzipped = typedArray.buffer;
    
    // Decode CBOR data first time:
    [headers1, headers2, cbor_data, signature] = CBOR.decode(unzipped);
    cbor_dataArr = typedArrayToBuffer(cbor_data);
    
    // Decode CBOR data second time:
    greenpassData  = CBOR.decode(cbor_dataArr);

--------------

Useful resources:

- Greenpass (Digital Covid Certificate / DGC) official JSON specification (PDF, v. 1.3.0, june 2021): https://ec.europa.eu/health/sites/default/files/ehealth/docs/covid-certificate_json_specification_en.pdf
- Original base45 node.js decoder: https://github.com/dirkx/base45-js/blob/main/lib/base45-js.js
- Zlib unpacker library for browser: https://github.com/nodeca/pako/blob/master/dist/pako.min.js
- CBOR unpacker: https://github.com/paroga/cbor-js
- CBOR decoder/viewer online: http://cbor.me/
- Dummy greenpass for testing: https://gir.st/blog/img/greenpass-demo.png
- Official data for testing: https://github.com/eu-digital-green-certificates/dgc-testdata/tree/main/IT/png
- Documentation on json fields: https://ec.europa.eu/health/sites/default/files/ehealth/docs/digital-green-certificates_dt-specifications_en.pdf
- Databases for decoding fields values: https://github.com/ehn-dcc-development/ehn-dcc-schema/tree/release/1.3.0/valuesets
- My reference guides for decoding greenpass: https://dev.to/lmillucci/javascript-how-to-decode-the-greenpass-qr-code-3dh0
- https://gir.st/blog/greenpass.html : Sample data containing 5 strings:
    - Original QR-decoded string
    - BASE45 string
    - Compressed string
    - COSE string
    - CBOR string
