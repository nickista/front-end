# front-end

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Drag and Drop Menu with Payment</title>
    <style>
        .menu-item, .cart-item { border: 1px solid #ccc; padding: 10px; margin: 5px; cursor: grab; }
        .admin-panel { display: none; margin-top: 20px; }
        .hidden { display: none; }
        .payment-method { display: none; margin-top: 20px; }
        .payment-button { margin: 5px; padding: 10px; cursor: pointer; }
    </style>
</head>
<body>
    <div id="main-content">
        <h1>Menu</h1>
        <div id="menu-items" style="display: flex; flex-wrap: wrap;"></div>
        <button id="toggle-admin">Admin Mode</button>
        <div class="admin-panel">
            <input type="text" id="menu-name" placeholder="Menu Name">
            <input type="number" id="menu-price" placeholder="Price">
            <input type="file" id="menu-image">
            <button id="add-menu">Add Menu</button>
        </div>
    </div>
    <div>
        <h1>Cart</h1>
        <div id="cart-items" style="border: 1px solid #000; padding: 10px; min-height: 100px;"></div>
        <p>Total: <span id="cart-total">0</span> 원</p>
        <button id="checkout">Checkout</button>
    </div>

    <div id="payment-method" class="payment-method">
        <h2>Choose Payment Method</h2>
        <button class="payment-button" data-method="cash">Cash</button>
        <button class="payment-button" data-method="card">Card</button>
        <button class="payment-button" data-method="app">App Payment</button>
    </div>

    <script>
        let menuItems = [
            { id: 1, name: "Americano", price: 4000, image: "images/americano.jpg" },
            { id: 2, name: "Latte", price: 4500, image: "images/latte.jpg" },
            { id: 3, name: "Cappuccino", price: 5000, image: "images/cappuccino.jpg" }
        ];

        let cartItems = [];

        function renderMenu() {
            const menuContainer = document.getElementById("menu-items");
            menuContainer.innerHTML = "";
            menuItems.forEach(item => {
                const div = document.createElement("div");
                div.classList.add("menu-item");
                div.setAttribute("draggable", true);
                div.dataset.id = item.id;
                div.innerHTML = `
                    <img src="${item.image}" alt="${item.name}" width="50">
                    <p>${item.name}</p>
                    <p>${item.price} 원</p>
                `;
                div.addEventListener("dragstart", handleDragStart);
                menuContainer.appendChild(div);
            });
        }

        function renderCart() {
            const cartContainer = document.getElementById("cart-items");
            cartContainer.innerHTML = "";
            let total = 0;
            cartItems.forEach(item => {
                const div = document.createElement("div");
                div.classList.add("cart-item");
                div.dataset.id = item.id;
                div.innerHTML = `
                    <img src="${item.image}" alt="${item.name}" width="50">
                    <p>${item.name}</p>
                    <p>${item.price} 원</p>
                    <div>
                        <button class="decrease">-</button>
                        <span>${item.quantity}</span>
                        <button class="increase">+</button>
                    </div>
                    <p>Subtotal: ${item.quantity * item.price} 원</p>
                `;
                total += item.quantity * item.price;
                div.querySelector(".increase").addEventListener("click", () => updateQuantity(item.id, 1));
                div.querySelector(".decrease").addEventListener("click", () => updateQuantity(item.id, -1));
                cartContainer.appendChild(div);
            });
            document.getElementById("cart-total").textContent = total;
        }

        function updateQuantity(id, delta) {
            const item = cartItems.find(item => item.id === id);
            if (!item) return;
            item.quantity += delta;
            if (item.quantity <= 0) {
                cartItems = cartItems.filter(item => item.id !== id);
            }
            renderCart();
        }

        function handleDragStart(event) {
            event.dataTransfer.setData("text/plain", event.target.dataset.id);
        }

        function handleDrop(event, area) {
            event.preventDefault();
            const id = parseInt(event.dataTransfer.getData("text/plain"));
            if (area === "cart") {
                const item = menuItems.find(item => item.id === id);
                if (item) {
                    menuItems = menuItems.filter(item => item.id !== id);
                    cartItems.push({ ...item, quantity: 1 });
                }
            } else if (area === "menu") {
                const item = cartItems.find(item => item.id === id);
                if (item) {
                    cartItems = cartItems.filter(item => item.id !== id);
                }
            }
            renderMenu();
            renderCart();
        }

        document.getElementById("cart-items").addEventListener("dragover", event => event.preventDefault());
        document.getElementById("cart-items").addEventListener("drop", event => handleDrop(event, "cart"));

        document.getElementById("menu-items").addEventListener("dragover", event => event.preventDefault());
        document.getElementById("menu-items").addEventListener("drop", event => handleDrop(event, "menu"));

        // Admin toggle and menu addition
        document.getElementById("toggle-admin").addEventListener("click", () => {
            const password = prompt("Enter admin password:");
            if (password === "admin") {
                document.querySelector(".admin-panel").classList.toggle("hidden");
            } else {
                alert("Incorrect password!");
            }
        });

        document.getElementById("add-menu").addEventListener("click", () => {
            const name = document.getElementById("menu-name").value;
            const price = parseInt(document.getElementById("menu-price").value);
            const imageInput = document.getElementById("menu-image");
            const image = imageInput.files[0] ? URL.createObjectURL(imageInput.files[0]) : "images/default.jpg";

            if (name && price) {
                const newItem = { id: Date.now(), name, price, image };
                menuItems.push(newItem);
                renderMenu();
            } else {
                alert("Please fill all fields!");
            }
        });

        // Checkout button functionality
        document.getElementById("checkout").addEventListener("click", () => {
            if (cartItems.length === 0) {
                alert("Your cart is empty!");
                return;
            }
            document.getElementById("main-content").classList.add("hidden");
            document.getElementById("payment-method").style.display = "block";
        });

        // Payment method selection
        document.querySelectorAll(".payment-button").forEach(button => {
    button.addEventListener("click", () => {
        const method = button.dataset.method;
        alert(`Payment via ${method} completed!`);

        // Clear the cart and render updates
        cartItems = [];
        renderCart();
        renderMenu(); // Ensure the menu is re-rendered

        // Hide payment screen and show the main content
        document.getElementById("payment-method").style.display = "none";
        document.getElementById("main-content").classList.remove("hidden");
    });
});


        renderMenu();
    </script>
</body>
</html>
