# greenpass
 Pure javascript greenpass QR code decoder in browser
 
 - Main page: <a href="http://jumpjack.altervista.org/greenpass/">link</a>
 - Test page for developers (only console output): <a href="https://jumpjack.altervista.org/greenpass/mini.html">link</a>
 - Qrcode library: https://github.com/klonikar/qrcodejs
 - Test data taken  from: https://github.com/eu-digital-green-certificates/dgc-testdata/tree/main/IT

--------------------

With this tool you can decode, view and understand all the data actually contained in your greenpass/QR, which are instead not visible using the standard app to check green pass validity, but you can't validate (i.e. verify if QR is valid and legal), this function is not implemented; but the script exposes variables **headers1, headers2, cbor_data,** and  **signature**; greenpass JSON data are in **cbor_data**, the other variables are for validation.

You don't need to upload your QR code to any site, all processing is performed offline: just drag the QR code on the specified page, or paste the string resulting from QR code reading by any app, then click a button and see all your data.

The decoding process is:

QR code --> **QR DECODER** --> RAW QR-decoded string --> **BASE45 decoder** --> zlib compressed string --> **pako library** --> COSE string --> **CBOR decoder** --> CBOR string --**> CBOR decoder** --> final JSON file

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

- Original base45 node.js decoder: https://github.com/dirkx/base45-js/blob/main/lib/base45-js.js
- Zlib unpacker library for browser: https://github.com/nodeca/pako/blob/master/dist/pako.min.js
- CBOR unpacker: https://github.com/paroga/cbor-js
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
