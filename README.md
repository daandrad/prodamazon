# prodamazon
// backend/index.ts
import express from "express";
import axios from "axios";
import { JSDOM } from "jsdom";

const app = express();
const PORT = 3000;

app.get("/api/scrape", async (req, res) => {
    const keyword = req.query.keyword;
    if (!keyword) {
        return res.status(400).json({ error: "Keyword is required" });
    }

    try {
        const url = `https://www.amazon.com/s?k=${encodeURIComponent(keyword)}`;
        const { data } = await axios.get(url, {
            headers: {
                "User-Agent": "Mozilla/5.0"
            }
        });

        const dom = new JSDOM(data);
        const document = dom.window.document;

        const products = [];
        const items = document.querySelectorAll(".s-main-slot .s-result-item");

        items.forEach(item => {
            const titleElement = item.querySelector("h2 a span");
            const ratingElement = item.querySelector(".a-icon-alt");
            const reviewsElement = item.querySelector(".a-size-base");
            const imageElement = item.querySelector(".s-image");

            if (titleElement && imageElement) {
                products.push({
                    title: titleElement.textContent.trim(),
                    rating: ratingElement ? ratingElement.textContent.split(" ")[0] : "N/A",
                    reviews: reviewsElement ? reviewsElement.textContent.trim() : "0",
                    imageUrl: imageElement.getAttribute("src")
                });
            }
        });

        res.json(products);
    } catch (error) {
        res.status(500).json({ error: "Failed to scrape Amazon" });
    }
});

app.listen(PORT, () => {
    console.log(`Server running at http://localhost:${PORT}`);
});

// frontend/main.js
document.getElementById("searchBtn").addEventListener("click", async () => {
    const keyword = document.getElementById("keywordInput").value;
    if (!keyword) return;

    const response = await fetch(`/api/scrape?keyword=${encodeURIComponent(keyword)}`);
    const products = await response.json();
    
    const resultsContainer = document.getElementById("results");
    resultsContainer.innerHTML = "";

    products.forEach(product => {
        const productDiv = document.createElement("div");
        productDiv.innerHTML = `
            <h3>${product.title}</h3>
            <p>Rating: ${product.rating} ⭐</p>
            <p>Reviews: ${product.reviews}</p>
            <img src="${product.imageUrl}" alt="Product Image" width="100">
            <hr>
        `;
        resultsContainer.appendChild(productDiv);
    });
});

// frontend/index.html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Amazon Scraper</title>
    <script defer src="./main.js"></script>
</head>
<body>
    <input type="text" id="keywordInput" placeholder="Enter keyword">
    <button id="searchBtn">Search</button>
    <div id="results"></div>
</body>
</html>

// README.md
# Amazon Scraper

## Configuração
1. Instale o Bun: [Bun](https://bun.sh/)
2. Clone o repositório e instale as dependências:
   ```sh
   bun install
   ```
3. Inicie o servidor:
   ```sh
   bun run backend/index.ts
   ```
4. Inicie o frontend com Vite:
   ```sh
   cd frontend && bun run dev
   ```
5. Acesse `http://localhost:5173` e faça buscas na Amazon.

## Considerações
- Use um proxy se necessário, pois a Amazon pode bloquear requisições frequentes.
- O projeto foi feito para fins educacionais.
