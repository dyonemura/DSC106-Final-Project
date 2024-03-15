<script>
  import { onMount, onDestroy } from "svelte";
  let animate = false;
  let animate2 = false;
  let isInView = false;
  const options = {};

  let count, index, offset, progress;

  //Text Code
  let text = "Password";
  let displayText = text;

  function setText() {
    displayText = text;
  }

  const moveWord = () => {
    const start = document.getElementById("start");
    const end = document.getElementById("end");
    const word = document.querySelector(".word");
    const section = document.querySelector("div");
    const sectionRect = section.getBoundingClientRect();
    const startPosition =
      start.getBoundingClientRect().right - sectionRect.left;
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
    }, 3000);
  };

  const moveWord2 = () => {
    const start = document.getElementById("start");
    const end = document.getElementById("end");
    const word = document.querySelector(".word");
    const section = document.querySelector("div");

    const sectionRect = section.getBoundingClientRect();
    const startPosition =
      start.getBoundingClientRect().right - sectionRect.left;
    const endPosition = end.getBoundingClientRect().left - sectionRect.left;

    const midPosition = (startPosition + endPosition) / 2;

    let position = startPosition;
    word.style.left = `${position}px`;

    setTimeout(() => {
      word.textContent = "Encrypted with key: " + simpleEncryptedText; // Change the word to the encrypted text
    }, 2000); // Wait for 2 seconds before changing the text

    setTimeout(() => {
      word.textContent = simpleEncryptedText; // Change the word to the encrypted text
    }, 4000); // Wait for 2 seconds before changing the text

    setTimeout(() => {
      position = position === startPosition ? midPosition : startPosition;
      word.style.left = `${position}px`;
      animate2 = true;

      setTimeout(() => {
        position = endPosition;
        word.style.left = `${position}px`;
      }, 4500); // Start the animation 2 seconds after changing the text
    }, 4750);
  };

  // Encryption Code
  let encryptedText = "";
  let secretKey_lst = [];
  let charCodeLst = [];
  let secretKey = "asjfbf712hrbajb";
  let encryptionLoop = [];
  let secretKeyText = secretKey.split("");

  let shift = 0;
  for (let i = 0; i < secretKey.length; i++) {
    shift += secretKey.charCodeAt(i);
    secretKey_lst.push(secretKey.charCodeAt(i));
  }

  let secretKeySum = secretKey_lst.reduce(
    (partialSum, currentNumber) => partialSum + currentNumber,
    0
  );
  shift = shift % 26;

  // Caesar Cypher
  function encrypt(text, shift) {
    charCodeLst = [];
    var result = "";
    for (var i = 0; i < text.length; i++) {
      var charCode = text.charCodeAt(i);
      charCodeLst.push(charCode);
      // Uppercase
      if (charCode >= 65 && charCode <= 90) {
        result += String.fromCharCode(((charCode - 65 + shift) % 26) + 65);
      } else if (charCode >= 97 && charCode <= 122) {
        result += String.fromCharCode(((charCode - 97 + shift) % 26) + 97);
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
      privateKey: [d, n],
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
    var messageNumbers = message
      .split("")
      .map((char) => BigInt(char.codePointAt(0)));
    var encryptedNumbers = messageNumbers.map((num) =>
      modPow(BigInt(num), e, n)
    );
    return encryptedNumbers;
  }

  function rsaDecrypt([d, n], encryptedMessage) {
    var decryptedNumbers = encryptedMessage.map((num) =>
      modPow(BigInt(num), d, n)
    );
    var message = decryptedNumbers
      .map((num) => String.fromCodePoint(Number(num)))
      .join("");
    return message;
  }

  $: simpleEncryptedText = encrypt(text, shift);
  $: encryptedText = rsaEncrypt(keys.publicKey, displayText);
  $: decryptedText = rsaDecrypt(keys.privateKey, encryptedText);

  // Animation Code

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

  encryptionLoop = [
    "Step 1 - Convert to Number: A = 1",
    "Step 2: Apply Shift - 1 + 3 = 4",
    "Step 3 - Convert Number to Letter: 4 = D",
  ];
  let currentElement = "";
  let indexxx = 0;
  let intervalId;

  onMount(() => {
    inview.observe(options);
  });

  function starter(encryptionLoop) {
    let intervalId = setInterval(() => {
      if (indexxx >= encryptionLoop.length) {
        clearInterval(intervalId);
      } else {
        let para = document.createElement("p");
        let node = document.createTextNode(encryptionLoop[indexxx]);
        para.appendChild(node);
        para.className = "fade";
        let element = document.getElementById("steps");
        element.appendChild(para);
        setTimeout(() => (para.className += " in"), 50); // Add 'in' class after a small delay
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
  // Done copying from stroll

  function clickOutside(element, callbackFunction) {
    function onClick(event) {
      if (!element.contains(event.target)) {
        callbackFunction();
      }
    }

    document.body.addEventListener("click", onClick);

    return {
      update(newCallbackFunction) {
        callbackFunction = newCallbackFunction;
      },
      destroy() {
        document.body.removeEventListener("click", onClick);
      },
    };
  }
  let bpressed = false;
  let one = false;
  let two = false;
  let three = false;
  let four = false;
  let five = false;
  let six = false;
  let intro = true;
</script>

{#if intro}
  <div class="laptop">
    <h1 class="maintitle">Visualize The Unseen</h1>
    <p style="color: white;  font-size:3rem">Bryan Gonzalez</p>
    <p style="color: white;  font-size:3rem">David Yonemura</p>
    <button
      class="introbutton"
      on:click={() => {
        intro = false;
      }}
    >
      Start
    </button>
  </div>
{:else if bpressed}
  {#if one}
    <div class="laptop2">
      <input
        bind:value={text}
        maxlength="5"
        style="width: 10rem; height: 2rem;  text-align:center; margin-right:10rem"
      />
      <button on:click={setText}>Set Password</button>
      <button
        class="goback"
        on:click={() => {
          bpressed = false;
          one = false;
        }}
        >Home Screen
      </button>
      <button
        class="next"
        on:click={() => {
          one = false;
          two = true;
        }}>Next</button
      >
    </div>
  {/if}
  {#if two}
    <button on:click={moveWord}>Send Text to Friend</button>
    <div class="laptop2">
      <div class="arrow" class:animate></div>
      <span>Hacker Intercepted</span>

      <div style="width: 100%; display: flex; justify-content: space-between;">
        <div id="start">You</div>
        <div id="end">Your Friend</div>
      </div>

      <div class="word" style="left: 0;">{text}</div>
      <button
        class="goback"
        on:click={() => {
          bpressed = false;
          two = false;
        }}
        >Home Screen
      </button>
      <button
        class="prev"
        on:click={() => {
          two = false;
          one = true;
        }}>Previous</button
      >
      <button
        class="next"
        on:click={() => {
          two = false;
          three = true;
        }}>Next</button
      >
    </div>
  {/if}
  {#if three}
    <div class="laptop2">
      <h1>{"Symmetric Key Encryption"}</h1>
      <p>
        {`You and your friend both have a randomly generated encryption key. This key is important as it contains the necessary information to decode the decrypted text. We will use a simple Caesar cypher to show the importance of a key.`}
      </p>
      <button
        class="goback"
        on:click={() => {
          bpressed = false;
          three = false;
        }}
        >Home Screen
      </button>
      <button
        class="prev"
        on:click={() => {
          three = false;
          two = true;
        }}>Previous</button
      >
      <button
        class="next"
        on:click={() => {
          three = false;
          four = true;
        }}>Next</button
      >
    </div>
  {/if}
  {#if four}
    <div class="laptop2">
      <h1>
        {"Symmetric Key Encryption: Caesar Cypher Example"}
      </h1>
      <p>
        {`Substitute each letter by shifting it down/up the alphabet.`}
      </p>
      <p>Letter to Encrypt: A; Shift 3</p>

      <button
        class="goback"
        on:click={() => {
          bpressed = false;
          four = false;
        }}
        >Home Screen
      </button>
      <button
        class="prev"
        on:click={() => {
          four = false;
          three = true;
        }}>Previous</button
      >
      <button
        class="next"
        on:click={() => {
          four = false;
          five = true;
        }}>Next</button
      >
    </div>
  {/if}
  {#if five}
    <div class="laptop2">
      <h1>{"Symmetric Key Encryption: Our Algorithm"}</h1>
      <p>{`You and your friend's key: ${secretKey}`}</p>
      <p>{secretKeyText}</p>
      <p>Convert each letter to its character code</p>
      <p>{secretKey_lst}</p>
      <p>Sum all values: {secretKeySum}</p>
      <p>
        Find remainder of summed values divided by 26, the number of letters in
        the alphabet
      </p>
      <p>Given the key, our shift value is: {shift}</p>
      <p>
        Your word:{text}. Your encrypted word: {simpleEncryptedText}
      </p>
      <button
        class="goback"
        on:click={() => {
          bpressed = false;
          five = false;
        }}
        >Home Screen
      </button>
      <button
        class="prev"
        on:click={() => {
          five = false;
          four = true;
        }}>Previous</button
      >
      <button
        class="next"
        on:click={() => {
          five = false;
          six = true;
        }}>Next</button
      >
    </div>
  {/if}
  {#if six}
    <div class="laptop2">
      <h1>With Symmetric Encryption</h1>

      <button on:click={moveWord2}>Send Text to Friend</button>
      <div class="arrow2" class:animate={animate2}></div>
      <span>Hacker Intercepted</span>

      <div style="width: 100%; display: flex; justify-content: space-between;">
        <div id="start">You</div>
        <div id="end">Your Friend</div>
      </div>

      <div class="word" style="left: 0;">{text}</div>
      <button
        class="goback"
        on:click={() => {
          bpressed = false;
          six = false;
        }}
        >Home Screen
      </button>
      <button
        class="prev"
        on:click={() => {
          six = false;
          five = true;
        }}>Previous</button
      >
    </div>
  {/if}
{:else}
  <div class="mainc">
    <div class="phone">
      <h1 style="color: white;">IMPORTANCE OF ENCRYPTION</h1>
      <div class="grid">
        <button
          class="homebutton1"
          on:click={(event) => {
            bpressed = true;
            one = true;
            event.stopPropagation();
          }}>Set your password</button
        >
        <button
          class="homebutton2"
          on:click={(event) => {
            bpressed = true;
            two = true;
            event.stopPropagation();
          }}>How hackers see your password without encryption</button
        >
        <button
          class="homebutton3"
          on:click={(event) => {
            bpressed = true;
            three = true;
            event.stopPropagation();
          }}>One algorithm</button
        >
        <button
          class="homebutton4"
          on:click={(event) => {
            bpressed = true;
            four = true;
            event.stopPropagation();
          }}>Example of the algorithm</button
        >
        <button
          class="homebutton5"
          on:click={(event) => {
            bpressed = true;
            five = true;
            event.stopPropagation();
          }}>The math behind the algorithm</button
        >
        <button
          class="homebutton6"
          on:click={(event) => {
            bpressed = true;
            six = true;
            event.stopPropagation();
          }}>How hackers see your password with encryption</button
        >
      </div>
    </div>
  </div>
{/if}

<style>
  .maintitle {
    font-size: 5rem;
  }
  .homebutton1 {
    border: black;
    outline: black;
    width: 100%;
    height: 100%;
    color: rgb(0, 0, 0);
    font-size: 5vmin;
    border: none;
    margin: 0;
    will-change: transform;
    background-image: linear-gradient(
      to right,
      rgb(255, 242, 0),
      rgba(228, 228, 16, 0.662)
    );
  }
  .homebutton2 {
    border: black;
    outline: black;
    width: 100%;
    height: 100%;
    color: rgb(0, 0, 0);
    font-size: 5vmin;
    border: none;
    margin: 0;
    will-change: transform;
    background-image: linear-gradient(
      to right,
      rgb(0, 170, 255),
      rgba(106, 211, 234, 0.662)
    );
  }
  .homebutton3 {
    border: black;
    outline: black;
    width: 100%;
    height: 100%;
    color: rgb(0, 0, 0);
    font-size: 5vmin;
    border: none;
    margin: 0;
    will-change: transform;
    background-image: linear-gradient(
      to right,
      rgb(255, 0, 68),
      rgba(234, 94, 94, 0.662)
    );
  }
  .homebutton4 {
    border: black;
    outline: black;
    width: 100%;
    height: 100%;
    color: rgb(0, 0, 0);
    font-size: 5vmin;
    border: none;
    margin: 0;
    will-change: transform;
    background-image: linear-gradient(
      to right,
      rgb(0, 255, 38),
      rgba(135, 255, 145, 0.662)
    );
  }
  .homebutton5 {
    border: black;
    outline: black;
    width: 100%;
    height: 100%;
    color: rgb(0, 0, 0);
    font-size: 5vmin;
    border: none;
    margin: 0;
    will-change: transform;
    background-image: linear-gradient(
      to right,
      rgb(195, 0, 255),
      rgba(142, 68, 127, 0.662)
    );
  }
  .homebutton6 {
    border: black;
    outline: black;
    width: 100%;
    height: 100%;
    color: rgb(0, 0, 0);
    font-size: 5vmin;
    border: none;
    margin: 0;
    will-change: transform;
    background-image: linear-gradient(
      to right,
      rgb(84, 3, 205),
      rgba(100, 72, 124, 0.662)
    );
  }
  .modal {
    background-color: rgba(68, 152, 225, 0.406);
    position: relative;
    display: flex;
    flex-direction: column;
    width: 100%;
    height: 100%;
    border: 2vmin solid #ccc;
    border-bottom-width: 10vmin;
    padding: 3vmin;
    border-radius: 2vmin;
  }
  .mainc {
    position: absolute;
    display: flex;
    align-items: center;
    justify-content: center;
    background-color: #01010ecb;
    width: 100%;
    height: 100%;
    top: 0;
    left: 0;
  }
  .intro {
    background-color: #01010ecb;
    width: 100vw;
    height: 100vh;
    padding-top: 0;
    margin-top: 0;
  }

  .phone {
    align-items: center;
    justify-content: center;
    background-color: #cccccc;
    position: relative;
    display: flex;
    flex-direction: column;
    width: 90%;
    height: 90%;
    border: 2vmin solid #000000;
    border-bottom-width: 3vmin;
    padding: 3vmin;
    border-radius: 2vmin;
    background-image: linear-gradient(rgb(106, 129, 243), rgb(184, 115, 189));
    background-size: cover;
  }
  .laptop {
    align-items: center;
    justify-content: center;
    background-image: linear-gradient(rgb(106, 129, 243), rgb(184, 115, 189));
    position: relative;
    display: flex;
    flex-direction: column;
    width: 90%;
    height: 40rem;
    border: 2vmin solid #000000;
    border-bottom-width: 3vmin;
    padding: 3vmin;
    border-radius: 2vmin;
    margin-left: 2rem;
    margin-top: 1rem;
    background-size: cover;
  }
  .laptop2 {
    align-items: center;
    justify-content: center;
    background-image: linear-gradient(rgb(106, 129, 243), rgb(184, 115, 189));
    position: relative;
    display: flex;
    flex-direction: column;
    width: 90%;
    height: 40rem;
    border: 2vmin solid #000000;
    border-bottom-width: 3vmin;
    padding: 3vmin;
    border-radius: 2vmin;
    margin-left: 2rem;
    margin-top: 1rem;
    background-size: cover;
  }
  .homebutton {
    background-color: aquamarine;
    border: black;
    outline: black;
    width: 100%;
    height: 100%;
    color: rgb(0, 0, 0);
    font-size: 5vmin;
    border: none;
    margin: 0;
    will-change: transform;
  }
  h1 {
    align-self: center;
  }
  .grid {
    display: grid;
    flex: 1;
    grid-template-columns: repeat(2, 1fr);
    grid-template-rows: repeat(3, 1fr);
    grid-gap: 1rem;
  }
  .goback {
    background-image: linear-gradient(
      to right,
      rgb(201, 186, 230),
      rgb(93, 125, 211)
    );
    width: 10rem;
    height: 3rem;
    position: absolute;
    bottom: 0;
    left: 45%;
    border-color: black;
  }
  .prev {
    background-image: linear-gradient(
      to right,
      rgb(226, 74, 47),
      rgb(240, 121, 57)
    );
    width: 10rem;
    height: 3rem;
    position: absolute;
    bottom: 0;
    left: 0%;
    border-color: black;
  }
  .next {
    background-image: linear-gradient(
      to right,
      rgb(82, 226, 154),
      rgb(35, 96, 97)
    );
    width: 10rem;
    height: 3rem;
    position: absolute;
    bottom: 0;
    right: 0%;
    border-color: black;
  }

  .word {
    position: absolute;
    transition:
      left 2s ease-in-out,
      opacity 1s;
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
    0% {
      border-top: 0 solid black;
    }
    100% {
      border-top: 40px solid black;
    }
  }

  .arrow2 {
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

  .arrow2.animate {
    display: block;
    animation: draw-arrow 2s forwards;
  }

  @keyframes draw-arrow2 {
    0% {
      border-top: 0 solid black;
    }
    100% {
      border-top: 40px solid black;
    }
  }

  .container {
    display: flex;
    align-items: center;
  }

  .container2 {
    display: flex;
    align-items: center;
  }

  span {
    margin-left: 67rem; /* Adjust as needed */
    opacity: 0;
    transition: opacity 2s;
  }

  .animate ~ span {
    animation: show-text 2s forwards;
    animation-delay: 2s;
  }

  @keyframes show-text {
    0% {
      opacity: 0;
    }
    100% {
      opacity: 1;
    }
  }

  .letter {
    display: inline-block;
    width: 1ch;
    height: 1em;
    overflow: hidden;
  }

  @keyframes letterChange {
    0% {
      opacity: 1;
    }
    50% {
      opacity: 0;
    }
    100% {
      opacity: 1;
    }
  }

  .fade {
    opacity: 0;
    transition: opacity 0.5s ease-in-out;
  }

  .fade.in {
    opacity: 1;
  }

  button {
    height: 2rem;
    width: 5rem;
    background-color: #aaa;
    border-color: #c644ea;
    color: #000000;
    font-size: 1.25rem;
    background-image: linear-gradient(45deg, #ec4b4b 50%, transparent 75%);
    background-position: 100%;
    background-size: 400%;
    transition: background 100ms ease-in-out;
  }
  button:hover {
    background-position: 0;
    color: #000000;
  }
  .introbutton {
    height: 3.3rem;
    width: 9rem;
    background-color: #ffffff;
    border-color: #68a0f5e2;
    color: #000000;
    font-size: 1.25rem;
    background-image: linear-gradient(45deg, #7c90ff 50%, transparent 50%);
    background-position: 100%;
    background-size: 400%;
    transition: background 300ms ease-in-out;
  }
</style>
