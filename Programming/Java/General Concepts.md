
## Registries
- [ Generally in Java ] A **registry** in Java is essentially a data structure or a system that maps **unique identifiers** (often strings) to specific objects. It allows you to register objects (like classes or instances) and then retrieve or interact with them later using their identifiers.

- [ In context of Minecraft Forge] In Minecraft, **registries** are used to register various game components (like blocks, items, entities, and more) so that they can be accessed, used, and manipulated by the game. These registries are part of Forge's system to handle the game's assets and behavior.

Deferred registry 

## Suppliers

In Java, a **`Supplier<T>`** is a functional interface from the `java.util.function` package. It represents a function that **takes no arguments and returns a result of type `T`**. It is typically used when you need to generate or supply values on demand.

@FunctionalInterface
public interface Supplier<T> {
    T get();
}

Example of Supplier: 
import java.util.function.Supplier;

public class SupplierExample {
    public static void main(String[] args) {
        Supplier<String> messageSupplier = () -> "Hello, Supplier!";
        System.out.println(messageSupplier.get());  // Output: Hello, Supplier!
    }
}

OR

// Registering an item using a Deferred Register in Forge
public static final RegistryObject<Item> SAPPHIRE = ITEMS.register("sapphire", () -> new Item(new Item.Properties()));


generate new data!

