<script>
  import Scroller from "@sveltejs/svelte-scroller";
  import { fade } from 'svelte/transition';

  let count, index, offset, progress;

  //Text Code
  let text = "Hello, Bob";
  let displayText = text;

  function setText() {
    displayText = text;
  }


  // Encryption Code
  let encryptedText = '';
  let encryptedLst = [];
  let charCodeLst = [];
  let secretKey = 'asjfbf712hrbajb';

  let shift = secretKey.length % 26;

  // Caesar Cypher
  function encrypt(text, shift) {
      charCodeLst = [];
      var result = '';
      for (var i = 0; i < text.length; i++) {
          var charCode = text.charCodeAt(i);
          charCodeLst.push(charCode)
          // Uppercase
          if (charCode >= 65 && charCode <= 90) {
              result += String.fromCharCode((charCode - 65 + shift) % 26 + 65);
          } else if (charCode >= 97 && charCode <= 122) { //Lowercase
              result += String.fromCharCode((charCode - 97 + shift) % 26 + 97);
          } else {
              result += text.charAt(i); //Neither
          }
      }
      return result;
      
  }

  //RSA Encryption

  // Function to calculate the greatest common divisor (GCD)
  function gcd(a, b) {
      if (b === 0) return a;
      return gcd(b, a % b);
  }

  // Function to calculate the totient (phi) of a number
  function totient(p, q) {
      return (p - 1) * (q - 1);
  }

  // Function to generate RSA key pair
  function generateRSAKeys(p, q) {
      const n = p * q;
      const phi = totient(p, q);
      let e = 2; // Public exponent

      // Finding e such that e and phi are coprime
      while (e < phi) {
          if (gcd(e, phi) === 1) break;
          e++;
      }

      let d = 1; 

      while ((d * e) % phi !== 1) {
          d++;
      }
      return {
          publicKey: [e, n],
          privateKey: [d, n]
      };
  }

  const p = 61;
  const q = 53; 

  const keys = generateRSAKeys(p, q);
  console.log("Public Key:", keys.publicKey);
  console.log("Private Key:", keys.privateKey);


  function modPow(base, exponent, modulus) {
    base = BigInt(base);
    exponent = BigInt(exponent);
    modulus = BigInt(modulus);
    let result = BigInt(1);
    base = base % modulus;
    while (exponent > 0) {
        if (exponent % BigInt(2) === BigInt(1))  // If exponent is odd
           result = (result * base) % modulus;
        exponent = exponent >> BigInt(1); // Equivalent to dividing by 2
        base = (base * base) % modulus;
    }
    return result;
}

function rsaEncrypt([e, n], message) {
    var messageNumbers = message.split('').map(char => BigInt(char.codePointAt(0)));
    var encryptedNumbers = messageNumbers.map(num => modPow(BigInt(num), e, n));
    return encryptedNumbers;
}

function rsaDecrypt([d, n], encryptedMessage) {
    var decryptedNumbers = encryptedMessage.map(num => modPow(BigInt(num), d, n));
    var message = decryptedNumbers.map(num => String.fromCodePoint(Number(num))).join('');
    return message;
}

$: encryptedText = rsaEncrypt(keys.publicKey, displayText);
$: decryptedText = rsaDecrypt(keys.privateKey, encryptedText);

  //$: encryptedLst = encryptedText.split("");
  //{#if index === 3}
  //     <p>
  //        {#each Array.from(displayText) as char (char)}
  //         <span transition:fade={{duration: 2000}}>{encrypt(char)}</span>
  //        {/each}
  //     </p>
  //    {/if}
  // Random comment to test

</script>

<style>
  .background {
    width: 100%;
    height: 100vh;
    position: relative;
    outline: green solid 3px;
  }

  .foreground {
    width: 50%;
    margin: 0 auto;
    height: auto;
    position: relative;
    outline: red solid 3px;
  }

  .progress-bars {
    position: absolute;
    background: rgba(170, 51, 120, 0.2) /*  40% opaque */;
    visibility: visible;
  }

  section {
    height: 80vh;
    background-color: rgba(0, 0, 0, 0.2); /* 20% opaque */
    /* color: white; */
    outline: magenta solid 3px;
    text-align: center;
    max-width: 750px; /* adjust at will */
    color: black;
    padding: 1em;
    margin: 0 0 2em 0;
  }

  

</style>

<Scroller
  top={0.0}
  bottom={1}
  threshold={0.5}
  bind:count
  bind:index
  bind:offset
  bind:progress
>
  <div class="background" slot="background">

    <div class="progress-bars">
      <p>current section: <strong>{index + 1}/{count}</strong></p>
      <progress value={count ? (index + 1) / count : 0} />

      <p>offset in current section</p>
      <progress value={offset || 0} />

      <p>total progress</p>
      <progress value={progress || 0} />
    </div>
  </div>

  <div class="foreground" slot="foreground">
    <section>
    <p transition:fade={{duration: 2000}}>{'Imagine you are communicating with your friend, Bob.'}</p>
      
    </section>
    <section>This is the second section.
      <input bind:value={text}/>
      <button on:click={setText}>Set Text</button>

    </section>
    <section> 
      {#if index === 2}
        <p transition:fade>{displayText}</p>
        <p transition:fade>{encryptedText}</p>
        <p transition:fade>{decryptedText}</p>
      {/if}
    </section>
    <section> 
      
    </section>
    <section> 
      {#if index === 4}

        bruh
      {/if}
    </section>
  </div>
</Scroller>