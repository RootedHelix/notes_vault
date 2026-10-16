
## Allowing variables or methods to allow the DOM to finish loading 

	Alternatively, you can wrap it in a `DOMContentLoaded` listener if you need it to run as soon as the DOM is ready:

`document.addEventListener('DOMContentLoaded', () => {   const playerCanvas = document.getElementById('playerCanvas');   // Additional code here });`