# Swift_Define_Your_Own_Lazy_Sequence

```swift
import UIKit
import Foundation

// Data Model
struct Model {
    var title: String = ""
}

// DataBase
class Database {
    static let shared = Database()
    
    func model(at: Int) -> Model? {
        // Query...
        return Model()
    }
    
}

// 1. Define Iterator
struct ModelIterator: IteratorProtocol {
    private let database: Database
    private var index: Int = 0

    init(database: Database = .shared) {
        self.database = database
    }

    // Returning nil from an iterator’s next() method signals to Swift that the sequence has come to an end and the iteration will stop.
    mutating func next() -> Model? {
        let model: Model? = self.database.model(at: self.index)
        self.index += 1
        return model
    }
}

// 2. Define Your Own Sequence
struct ModelSequence: Sequence {
    func makeIterator() -> ModelIterator {
        return ModelIterator()
    }
}


// Practical Usage - Lazily search for model
// The nice thing about this approach, is that as soon as we’ve found a match, we can simply exit out of the iteration by returning, preventing further database records from being loaded.
func searchForModel(matching query: String) -> Model? {
    for model in ModelSequence() {
        if model.title.contains(query) {
            return model
        }
    }
    return nil
}


/*
 when you are really lazy, the standard library’s AnySequence type has a closure-based API that you can use to quickly implement simple sequences, like this:
 */
class ModelLoader {
    
    private let database: Database

    init(database: Database = .shared) {
        self.database = database
    }
    
    func loadAllModels() -> AnySequence<Model> {
        return AnySequence { [weak self] () -> AnyIterator<Model> in
            var index: Int = 0
            return AnyIterator {
                let model: Model? = self?.database.model(at: index)
                index += 1
                return model
            }
        }
    }
}

```
