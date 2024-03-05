<script>
  import Scroller from "@sveltejs/svelte-scroller";
  import { fade } from 'svelte/transition';
  import { inview } from 'svelte-inview';

  import { onMount, onDestroy } from 'svelte';
  let animate = false;

  

  let isInView = false;
  const options = {};

  let count, index, offset, progress;

  //Text Code
  let text = "Hello";
  let displayText = text;

  function setText() {
    displayText = text;
  }

  const moveWord = () => {
    const start = document.getElementById('start');
    const end = document.getElementById('end');
    const word = document.querySelector('.word');
    const section = document.querySelector('section');

    const sectionRect = section.getBoundingClientRect();
    const startPosition = start.getBoundingClientRect().right - sectionRect.left;
    const endPosition = end.getBoundingClientRect().left - sectionRect.left;

    const midPosition = (startPosition + endPosition) / 2;

    let position = startPosition; 
    word.style.left = `${position}px`; 

    position = position === startPosition ? midPosition : startPosition;
    word.style.left = `${position}px`;
    animate = true;
    setTimeout(() => {
    position = endPosition;
    word.style.left = `${position}px`;
  }, 3000)
  }

  // Encryption Code
  let encryptedText = '';
  let secretKey_lst = [];
  let charCodeLst = [];
  let secretKey = 'asjfbf712hrbajb';
  let encryptedCharCode = [];
  let encryptionLoop = [];

  let shift = 0;
  for (let i = 0; i < secretKey.length; i++) {
      shift += secretKey.charCodeAt(i);
      secretKey_lst.push(secretKey.charCodeAt(i))
  }
  shift = shift % 26;

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
          } else if (charCode >= 97 && charCode <= 122) {
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
        if (exponent % BigInt(2) === BigInt(1))  
           result = (result * base) % modulus;
        exponent = exponent >> BigInt(1); 
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


$: simpleEncryptedText = encrypt(text, shift)
$: encryptedText = rsaEncrypt(keys.publicKey, displayText);
$: decryptedText = rsaDecrypt(keys.privateKey, encryptedText);


import { tick } from 'svelte';
  
  $: loo = text;
  $: target = encrypt(text, shift);
  $: currentWord = loo;
  $: currentLetterChar = 0;
  $: equation = [];
  $: currentEncryptedChar = 0;


  function isUppercase(char) {
    return char === char.toUpperCase();
}
function isLowercase(char) {
    return char === char.toUpperCase();
}

  function replaceOneLetterAtATime() {
    const wordArray = currentWord.split("");
    const targetArray = target.split("");

    for (let i = 0; i < wordArray.length; i++) {
      const currentLetter = wordArray[i];
      currentLetterChar = charCodeLst[i];

      if (isUppercase(currentLetter)) {
        equation[i] = `((${currentLetterChar} - 65 + ${shift}) % 26) + 65`;
      }
      if (isLowercase(currentLetter)) {
        equation[i] = `((${currentLetterChar} - 97 + ${shift}) % 26) + 97`;
      }


      currentEncryptedChar = simpleEncryptedText[i];

      if (wordArray[i] !== targetArray[i]) {
        wordArray[i] = targetArray[i];
        currentWord = wordArray.join("");
        break; // Replace only one letter at a time
      }
    }
  }

  // Simulate automatic replacement
  const interval = 2000; // 1 second
  let timer;

  function startReplacement() {
    timer = setInterval(replaceOneLetterAtATime, interval);
  }

  function stopReplacement() {
    clearInterval(timer);
  }

  // Start the replacement process when the component is mounted
  
  encryptionLoop = ["Initial Letter: A", "Convert to Number: 1", '1 (A) + 3 (The Shift)', "Shifted Number: 4", "Final Letter: D"];
  let currentElement = encryptionLoop[0];
  let indexxx = 0;
  let intervalId;

  onMount(() => {
    inview.observe(options);
  });

  function starter(encryptionLoopoop) {
    let intervalId = setInterval(() => {
      if (indexxx >= encryptionLoop.length) {
        clearInterval(intervalId);
      } else {
        currentElement = encryptionLoop[indexxx];
        indexxx++;
      }
    }, 5000);
  }

  $: if (isInView) {
    startReplacement();
    starter(encryptionLoop);
    
    
  } else if (intervalId) {
    clearInterval(intervalId);
  }

  onDestroy(() => {
    if (intervalId) {
      clearInterval(intervalId);
    }
  });

</script>

<style>
  .container {
    display: flex;
    align-items: center;
  }
  .word {
    position: absolute;
    transition: all 2s ease-in-out;
  }
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
    word-wrap: break-word;
    position: relative;
  }

  .arrow {
    display: none;
    width: 0;
    height: 0;
    border-left: 20px solid transparent;
    border-right: 20px solid transparent;
    border-top: 40px solid black;
    transition: border-top 2s;
    position: relative;
    left: 50%;
    transform: translateX(-50%);
  }

  .arrow.animate {
    display: block;
    animation: draw-arrow 2s forwards;
  }

  @keyframes draw-arrow {
    0% { border-top: 0 solid black; }
    100% { border-top: 40px solid black; }
  }

  .container {
    display: flex;
    align-items: center;
  }

  span {
    margin-left: 360px; /* Adjust as needed */
    opacity: 0;
    transition: opacity 2s;
  }

  .animate ~ span {
    animation: show-text 2s forwards;
    animation-delay: 2s;
  }

  @keyframes show-text {
    0% { opacity: 0; }
    100% { opacity: 1; }
  }

  .letter {
    display: inline-block;
    width: 1ch;
    height: 1em;
    overflow: hidden;
  }

  @keyframes letterChange {
    0% { opacity: 1; }
    50% { opacity: 0; }
    100% { opacity: 1; }
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

    <!--Section 1-->
    <section>
      <p transition:fade={{duration: 2000}}>{"Five Character Limit"}</p>
      <input bind:value={text} maxlength="5"/>
      <button on:click={setText}>Set Text</button>
      
    </section>

    <!--Section 2-->
    <section>
      {#if index === 1}
      <h1>Without Encryption</h1>

      <button on:click={moveWord}>Send Text to Friend</button>
      <p transition:fade>{"Spy"}</p>

      <div class="container">
        <div class="arrow" class:animate={animate}></div>
        <span>Intercepted</span>
      </div>

      <div style="width: 100%; display: flex; justify-content: space-between;">
        <div id="start">You</div>
        <div id="end">Your Friend</div>
      </div>
    
      <div class="word" style="left: 0;">{text}</div>

      {/if}
      <p transition:fade={{duration: 2000}}>{""}</p>

    </section>

    <!--Section 3-->
    <section> 
      {#if index === 2}
        <h1 transition:fade>{"Symmetric Key Encryption"}</h1>
        <p transition:fade>{`You and your friend both have a randomly generated encryption key given to you by a messaging app for decoding the message: ${secretKey}`}</p>
        <p transition:fade>{`An example encryption algorithm: Caesar Shift`}</p>
        <p transition:fade>{`Substitute each letter by shifting it down/up the alphabet.`}</p>

        <div use:inview={options}
          on:inview_change={(event) => {
            const { inView, entry, scrollDirection, observer, node} = event.detail;
            isInView = inView;
          }}
          on:inview_enter={(event) => {
            const { inView, entry, scrollDirection, observer, node} = event.detail;
            isInView = inView;
          }}
          on:inview_leave={(event) => {
            const { inView, entry, scrollDirection, observer, node} = event.detail;
            isInView = inView;
          }}
          on:inview_init={(event) => {
            const { observer, node } = event.detail;
          }}>{isInView ? currentElement : 'Bye, Bye'}
      </div>




        
        
        
      {/if}

      
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

    </section>
    <section> 

    </section>
    <section> 

    </section>
    <section> 

    </section>
    <section> 

    </section>
    <section> 

    </section>
    <section> 

    </section>
    <section> 

    </section>
    <section> 

    </section>
    <section> 

    </section>
    <section> 

    </section>
    <section> 

    </section>
    <section> 

    </section>
    <section> 

    </section>
    <section> 

    </section>
    <section> 

    </section>
    <section> 

    </section>
    <section> 

    </section>
    <section> 

    </section>
    <section> 

    </section>
  </div>
</Scroller>