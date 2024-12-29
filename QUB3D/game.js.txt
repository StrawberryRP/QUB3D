// game.js

// Set up the grid dimensions and variables
const cols = 10;
const rows = 20;
let grid = [];
let currentBlock = null;
let score = 0;

// Initialize the grid
function createGrid() {
    const gridElement = document.getElementById("grid");
    for (let i = 0; i < rows * cols; i++) {
        const cell = document.createElement("div");
        cell.classList.add("grid-cell");
        grid.push(cell);
        gridElement.appendChild(cell);
    }
}

// Create a random block
function createBlock() {
    const shapes = [
        [[1, 1], [1, 1]], // O shape
        [[1, 1, 1], [0, 1, 0]], // T shape
        [[1, 1, 0], [0, 1, 1]], // Z shape
        [[0, 1, 1], [1, 1, 0]], // S shape
        [[1, 0, 0], [1, 1, 1]], // L shape
        [[0, 0, 1], [1, 1, 1]], // J shape
        [[1, 1, 1, 1]] // I shape
    ];
    const shape = shapes[Math.floor(Math.random() * shapes.length)];
    currentBlock = {
        shape: shape,
        x: Math.floor(cols / 2) - Math.floor(shape[0].length / 2),
        y: 0,
        color: getRandomColor()
    };
}

// Get a random color
function getRandomColor() {
    const colors = ["#FF6347", "#32CD32", "#1E90FF", "#FFD700", "#FF4500"];
    return colors[Math.floor(Math.random() * colors.length)];
}

// Draw the current block on the grid
function drawBlock() {
    for (let row = 0; row < currentBlock.shape.length; row++) {
        for (let col = 0; col < currentBlock.shape[row].length; col++) {
            if (currentBlock.shape[row][col] === 1) {
                const index = (currentBlock.y + row) * cols + (currentBlock.x + col);
                grid[index].style.backgroundColor = currentBlock.color;
            }
        }
    }
}

// Clear the current block from the grid
function clearBlock() {
    for (let row = 0; row < currentBlock.shape.length; row++) {
        for (let col = 0; col < currentBlock.shape[row].length; col++) {
            if (currentBlock.shape[row][col] === 1) {
                const index = (currentBlock.y + row) * cols + (currentBlock.x + col);
                grid[index].style.backgroundColor = "#444";
            }
        }
    }
}

// Move the block down
function moveDown() {
    clearBlock();
    currentBlock.y++;
    if (collision()) {
        currentBlock.y--;
        lockBlock();
        checkLines();
        createBlock();
    }
    drawBlock();
}

// Check if the block collides with other blocks or the floor
function collision() {
    for (let row = 0; row < currentBlock.shape.length; row++) {
        for (let col = 0; col < currentBlock.shape[row].length; col++) {
            if (currentBlock.shape[row][col] === 1) {
                const x = currentBlock.x + col;
                const y = currentBlock.y + row;
                if (x < 0 || x >= cols || y >= rows || grid[y * cols + x].style.backgroundColor !== "rgb(68, 68, 68)") {
                    return true;
                }
            }
        }
    }
    return false;
}

// Lock the block into place and update the grid
function lockBlock() {
    for (let row = 0; row < currentBlock.shape.length; row++) {
        for (let col = 0; col < currentBlock.shape[row].length; col++) {
            if (currentBlock.shape[row][col] === 1) {
                const x = currentBlock.x + col;
                const y = currentBlock.y + row;
                grid[y * cols + x].style.backgroundColor = currentBlock.color;
            }
        }
    }
}

// Check and clear complete lines
function checkLines() {
    for (let row = 0; row < rows; row++) {
        let isFull = true;
        for (let col = 0; col < cols; col++) {
            if (grid[row * cols + col].style.backgroundColor === "rgb(68, 68, 68)") {
                isFull = false;
                break;
            }
        }
        if (isFull) {
            clearLine(row);
            shiftLinesDown(row);
            score += 10;
            document.getElementById("score-value").textContent = score;
        }
    }
}

// Clear a completed line
function clearLine(row) {
    for (let col = 0; col < cols; col++) {
        grid[row * cols + col].style.backgroundColor = "#444";
    }
}

// Shift lines down after clearing a line
function shiftLinesDown(row) {
    for (let i = row; i > 0; i--) {
        for (let col = 0; col < cols; col++) {
            grid[i * cols + col].style.backgroundColor = grid[(i - 1) * cols + col].style.backgroundColor;
        }
    }
}

// Rotate the current block
function rotateBlock() {
    clearBlock();
    const rotatedShape = currentBlock.shape[0].map((_, index) => currentBlock.shape.map(row => row[index])).reverse();
    currentBlock.shape = rotatedShape;
    if (collision()) {
        currentBlock.shape = rotatedShape.reverse();
    }
    drawBlock();
}

// Move the block left
function moveLeft() {
    clearBlock();
    currentBlock.x--;
    if (collision()) {
        currentBlock.x++;
    }
    drawBlock();
}

// Move the block right
function moveRight() {
    clearBlock();
    currentBlock.x++;
    if (collision()) {
        currentBlock.x--;
    }
    drawBlock();
}

// Drop the block
function dropBlock() {
    while (!collision()) {
        clearBlock();
        currentBlock.y++;
        drawBlock();
    }
    currentBlock.y--;
    lockBlock();
    checkLines();
    createBlock();
}

// Event listeners for controls
document.getElementById("rotate-btn").addEventListener("click", rotateBlock);
document.getElementById("move-left-btn").addEventListener("click", moveLeft);
document.getElementById("move-right-btn").addEventListener("click", moveRight);
document.getElementById("drop-btn").addEventListener("click", dropBlock);

// Start the game
createGrid();
createBlock();
setInterval(moveDown, 500);
