# odaniloluna.github.io
<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Scroll Infinito</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 0;
            padding: 0;
            text-align: center;
        }
        .content {
            width: 80%;
            margin: 0 auto;
        }
        .item {
            border: 1px solid #ccc;
            margin: 10px;
            padding: 20px;
            background-color: #f9f9f9;
        }
        .loader {
            text-align: center;
            margin: 20px 0;
        }
    </style>
</head>
<body>
    <div class="content" id="content">
        <!-- Conteúdo inicial -->
        <div class="item">Item 1</div>
        <div class="item">Item 2</div>
        <div class="item">Item 3</div>
    </div>
    <div class="loader" id="loader">Carregando...</div>

    <script>
        let page = 1;
        const content = document.getElementById('content');
        const loader = document.getElementById('loader');

        const loadMoreContent = () => {
            loader.style.display = 'block';
            fetch(`https://jsonplaceholder.typicode.com/posts?_page=${page}&_limit=5`)
                .then(response => response.json())
                .then(data => {
                    data.forEach(item => {
                        const div = document.createElement('div');
                        div.classList.add('item');
                        div.textContent = item.title;
                        content.appendChild(div);
                    });
                    loader.style.display = 'none';
                })
                .catch(error => console.error('Erro ao carregar mais conteúdo:', error));
        };

        window.addEventListener('scroll', () => {
            if (window.innerHeight + window.scrollY >= document.body.offsetHeight - 100) {
                page++;
                loadMoreContent();
            }
        });

        // Carregar conteúdo inicial
        loadMoreContent();
    </script>
</body>
</html>
