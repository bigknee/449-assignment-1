    //update book
    @PutMapping("/books/{id}")
    public Book updateBook(@PathVariable Long id, @RequestBody Book updatedBook) {
        Book foundBook = books.stream()
                        .filter(book -> book.getId().equals(id))
                        .findFirst()
                        .orElse(null);

                if (foundBook != null) {
                    foundBook.setTitle(updatedBook.getTitle());
                    foundBook.setAuthor(updatedBook.getAuthor());
                    foundBook.setPrice(updatedBook.getPrice());
                }

                return foundBook;
    }


    //partial update
    @PatchMapping("/books/{id}")
    public Book updateBook(@PathVariable Long id, @RequestBody Map<String, Object> updates) {
        Book book = books.stream()
                .filter(b -> b.getId().equals(id))
                .findFirst()
                .orElse(null);

        if (book == null) return null;

        if (updates.containsKey("title")) {
            book.setTitle((String) updates.get("title"));
        }

        if (updates.containsKey("author")) {
            book.setAuthor((String) updates.get("author"));
        }
        if (updates.containsKey("price")) {
            book.setPrice(((Number) updates.get("price")).doubleValue());
        }

        return book;
    }

    //remove book
    @DeleteMapping("/books/{id}")
    public Book removeBook(@PathVariable Long id) {
        Book foundBook = books.stream()
                .filter(book -> book.getId().equals(id))
                .findFirst()
                .orElse(null);
        books.removeIf(book -> book.getId().equals(id));
        return foundBook;
    }

    //get endpoint with pagination
    @GetMappin
    public List<Book> getBooks(@RequestParam(defaultValue = "0") int page, @RequestParam(defaultValue = "10") int size){
        return books.stream()
                    .skip(page * size)
                    .limit(size)
                    .collect(Collectors.toList());
    }

    //Advanced GET endpoint with filtering, sorting, and pagination combined in the valid order
    @GetMapping("/books/paginated")
    public List<Book> sortBooksAdvanced(
            @RequestParam(required = false) String title,
            @RequestParam(required = false) String author,
            @RequestParam(required = false) Double minPrice,
            @RequestParam(required = false) Double maxPrice,

            @RequestParam(defaultValue = "title") String sortBy,
            @RequestParam(defaultValue = "asc") String order,

            @RequestParam(defaultValue = "0") int page,
            @RequestParam(defaultValue = "10") int size
    ) {

        Comparator<Book> comparator;
        switch(sortBy.toLowerCase()) {
            case "author":
                comparator = Comparator.comparing(Book::getAuthor);
                break;
            case "title":
                comparator = Comparator.comparing(Book::getTitle);
                break;
            default:
                comparator = Comparator.comparing(Book::getTitle);
                break;
        }

        if("desc".equalsIgnoreCase(order)) {
            comparator = comparator.reversed();
        }

        return books.stream()
                .filter(book -> {
                    boolean matchesTitle = title == null || book.getTitle().toLowerCase().contains(title.toLowerCase());
                    boolean matchesAuthor = author == null || book.getAuthor().toLowerCase().contains(author.toLowerCase());
                    boolean matchesMinPrice = minPrice == null || book.getPrice() >= minPrice;
                    boolean matchesMaxPrice = maxPrice == null || book.getPrice() <= maxPrice;

                    return matchesTitle && matchesAuthor && matchesMinPrice && matchesMaxPrice;
                })
                .sorted(comparator)
                .skip(page * size)
                .limit(size)
                .collect(Collectors.toList());
    }
