# QR_Code_Generator

# **How to Build an AI QR Code Generator using Next.js and Replicate's qrcode-stable-diffusion Model**

Get ready to dive into the world of QR code art with the help of AI! In this blog, we'll show you how to build an AI QR code generator using Next.js and Replicate's innovative qrcode-stable-diffusion model.

## **Introducing qrcode-stable-diffusion Model**

The qrcode-stable-diffusion model is the secret sauce to creating mesmerizing QR code artwork. Trained on a dataset of 150,000 QR code + QR code artwork couples, it strikes the perfect balance between aesthetics and the essential QR code shape.

Imagine a QR code that magically turns into a 3D anime-style artwork when scanned. With this model, we can blend prompts like "The French countryside, vivid colors, animation by Studio Ghibli" with web URLs like "[**https://replicate.com**](https://replicate.com/)" to generate QR code masterpieces that enchant everyone.

## **Prerequisites**

To follow along with this tutorial, you should have the following prerequisites:

1. Node.js and npm (Node Package Manager) installed on your machine.
2. Basic knowledge of JavaScript and React.
3. An active Replicate account and an API token. If you don't have one, you can sign up for an account on the [Replicate website](https://replicate.com/facebookresearch/musicgen).

## **Setting Up the Next.js Project**

Let's start by setting up a new Next.js project. Open your terminal and execute the following commands:

```bash
npx create-next-app ai-qrcode-generator

✔ Would you like to use TypeScript with this project? … No
✔ Would you like to use ESLint with this project? … Yes
✔ Would you like to use Tailwind CSS with this project? … Yes
✔ Would you like to use `src/` directory with this project? … Yes
✔ Use App Router (recommended)? … No
✔ Would you like to customize the default import alias? … No
```

This will create a new Next.js project in a directory named **`ai-qrcode-generator`**. Next, open the project in your preferred code editor.

```bash
cd ai-qrcode-generator
```

## **Installing Dependencies**

Next, we need to install the required dependencies for our project. In the terminal, navigate to the project directory and run the following command:

```bash
npm install replicate
```

Now, create a new file called **`.env`** in the root of your project and add the following environment variables:

```sql
REPLICATE_API_TOKEN=<paste-your-token-here>
```

Retrieve your API token from your [Replicate account settings](https://replicate.com/account/api-tokens).

## **Creating the Backend**

Now, let's explore the realm of QR code artistry. Create a new file named **`qr-generator.js`** inside the **`src/pages/api`** directory. This file will house the code for generating AI QR codes.

```jsx
import Replicate from "replicate";

export default async function handler(req, res) {
  const replicate = new Replicate({
    auth: process.env.REPLICATE_API_TOKEN,
  });

  console.log(req.body.prompt);
  console.log(req.body.qr_code_content);
  try {
    const output = await replicate.run(
      "nateraw/qrcode-stable-diffusion:9cdabf8f8a991351960c7ce2105de2909514b40bd27ac202dba57935b07d29d4",
      {
        input: {
          prompt: req.body.prompt || "...",
          qr_code_content: req.body.qr_code_content || "",
        },
      }
    );

    res.status(200).json({ qr_code: output });
  } catch (error) {
    console.error("AI QR code generation failed:", error);
    res.status(500).json({ error: "AI QR code generation failed" });
  }
}
```

The code sets up a handler function to initialize the Replicate client using your API token. It then calls the **`run`** method to generate an AI-infused QR code based on the provided prompts and QR code content.

## **Creating the Frontend**

Let's add a QR code art studio to our frontend. Open the **`pages/index.js`** file and replace its content with the following code:

```jsx
import { useState } from "react";

export default function Home() {
  const [qrCode, setQrCode] = useState("");
  const [prompt, setPrompt] = useState("");
  const [qrCodeContent, setQrCodeContent] = useState("");
  const [isLoading, setIsLoading] = useState(false);

  const generateQrCode = async () => {
    setIsLoading(true);

    try {
      const response = await fetch("/api/qr-generator", {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify({ prompt, qr_code_content: qrCodeContent }),
      });

      const { qr_code } = await response.json();
      setQrCode(qr_code);
    } catch (error) {
      console.error("Failed to generate QR code:", error);
    }

    setIsLoading(false);
  };

  return (
    <div className="flex flex-col items-center justify-center h-screen">
      <h1 className="text-3xl font-bold mb-6">AI QR Code Generator</h1>
      <input
        type="text"
        value={prompt}
        onChange={(e) => setPrompt(e.target.value)}
        placeholder="Enter your QR code artwork prompt"
        className="px-4 py-2 text-black border border-gray-300 rounded-lg mb-4 w-80 text-center"
      />
      <input
        type="text"
        value={qrCodeContent}
        onChange={(e) => setQrCodeContent(e.target.value)}
        placeholder="Enter the web URL for the QR code"
        className="px-4 py-2 text-black border border-gray-300 rounded-lg mb-4 w-80 text-center"
      />
      <button
        onClick={generateQrCode}
        className="px-4 py-2 bg-blue-500 text-white rounded-lg hover:bg-blue-600"
        disabled={isLoading}
      >
        {isLoading ? "Generating..." : "Generate QR Code"}
      </button>
      {isLoading && (
        <div className="mt-4">
          <div className="animate-spin rounded-full h-8 w-8 border-t-2 border-blue-500"></div>
        </div>
      )}
      {qrCode && (
        <div className="mt-4">
          <img src={qrCode} alt="Generated QR Code" />
        </div>
      )}
    </div>
  );
}
```

1. We use the **`useState`** hook from React to manage state variables: **`qrCode`** to store the generated QR code image URL, **`prompt`** to capture the QR code artwork prompt, **`qrCodeContent`** to capture the web URL for the QR code, and **`isLoading`** to indicate when the QR code is being generated.
2. When the "Generate QR Code" button is clicked, the **`generateQrCode`** function is triggered. It sets **`isLoading`** to **`true`**, indicating that the QR code generation process has started.
3. The function sends a POST request to the **`/api/qr-generator`** endpoint with the **`prompt`** and **`qrCodeContent`** as part of the request body.
4. Once the response is received, the QR code image URL is extracted from the JSON response and stored in the **`qrCode`** state variable.
5. If an error occurs during the generation process, an error message is logged to the console.
6. The **`isLoading`** state is set back to **`false`** after the generation process is complete.

A few example prompts:

1. prompt - **The modern cyberpunk cars, shrooms and skyscrapers, machines, lavendar color, animation by mappa studios**
QR code content - https://google.com
2. prompt - **The french countryside, green pastures, lush environment, vivid colors, animation by studio ghibli**
QR code content - https://youtube.com

## Conclusion

With Next.js and Replicate's qrcode-stable-diffusion model, we've crafted a virtual QR code art studio. Enter prompts, add web URLs, click "Generate QR Code," and voilà! Captivating 3D anime-style QR code art awaits.

From the French countryside to Studio Ghibli animations, we've transformed QR codes into artistic masterpieces. Scan these QR codes, and the magic unfolds with web URLs that come alive in artful splendor.

So, let your imagination run wild, share your QR code art with the world, and let Next.js and Replicate be your QR code art maestros. Get ready to dazzle your audience with AI-infused QR code brilliance. Happy QR code artistry!
