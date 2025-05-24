# LAB3_EDA
import java.util.*;
import java.io.*;

class Game {
    private String name;
    private String category;
    private int price;
    private int quality;
    public Game(String name, String category, int price, int quality) {
        this.name = name;
        this.category = category;
        this.price = price;
        this.quality = quality;
    }
    public String getName() { return name; }
    public String getCategory() { return category; }
    public int getPrice() { return price; }
    public int getQuality() { return quality; }
    public static Comparator<Game> byPrice = Comparator.comparingInt(Game::getPrice);
    public static Comparator<Game> byCategory = Comparator.comparing(Game::getCategory);
    public static Comparator<Game> byQuality = Comparator.comparingInt(Game::getQuality);
}
class Dataset {
    private ArrayList<Game> data;
    private String sortedByAttribute;
    private Random random = new Random();
    public Dataset(ArrayList<Game> data) {
        this.data = new ArrayList<>(data);
        this.sortedByAttribute = "none";
    }
    public enum SearchMethod {
        BY_PRICE("getGamesByPrice", "price"),
        BY_PRICE_RANGE("getGamesByPriceRange", "price"),
        BY_CATEGORY("getGamesByCategory", "category"),
        BY_QUALITY("getGamesByQuality", "quality");
        private final String methodName;
        private final String sortAttribute;
        SearchMethod(String methodName, String sortAttribute) {
            this.methodName = methodName;
            this.sortAttribute = sortAttribute;
        }
    }
    public enum SortAlgorithm {
        BUBBLE_SORT("bubbleSort"),
        INSERTION_SORT("insertionSort"),
        SELECTION_SORT("selectionSort"),
        MERGE_SORT("mergeSort"),
        QUICK_SORT("quickSort"),
        COUNTING_SORT("countingSort"),
        COLLECTIONS_SORT("collectionsSort");
        private final String algorithmName;
        SortAlgorithm(String algorithmName) {
            this.algorithmName = algorithmName;
        }
    }
    public ArrayList<Game> getGamesByPrice(int price) {
        ArrayList<Game> result = new ArrayList<>();
        if (sortedByAttribute.equals("price")) {
            int index = Collections.binarySearch(data, new Game("", "", price, 0), Game.byPrice);
            if (index >= 0) {
                addAdjacentMatches(result, index, game -> game.getPrice() == price);
            }
        } else {
            linearSearch(result, game -> game.getPrice() == price);
        }
        return result;
    }
    public ArrayList<Game> getGamesByPriceRange(int lowerPrice, int higherPrice) {
        ArrayList<Game> result = new ArrayList<>();
        if (sortedByAttribute.equals("price")) {
            binarySearchRange(result, lowerPrice, higherPrice, Game::getPrice);
        } else {
            linearSearch(result, game -> game.getPrice() >= lowerPrice && game.getPrice() <= higherPrice);
        }
        return result;
    }
    public ArrayList<Game> getGamesByCategory(String category) {
        ArrayList<Game> result = new ArrayList<>();
        if (sortedByAttribute.equals("category")) {
            int index = Collections.binarySearch(data, new Game("", category, 0, 0), Game.byCategory);
            if (index >= 0) {
                addAdjacentMatches(result, index, game -> game.getCategory().equals(category));
            }
        } else {
            linearSearch(result, game -> game.getCategory().equals(category));
        }
        return result;
    }
    public ArrayList<Game> getGamesByQuality(int quality) {
        ArrayList<Game> result = new ArrayList<>();
        if (sortedByAttribute.equals("quality")) {
            int index = Collections.binarySearch(data, new Game("", "", 0, quality), Game.byQuality);
            if (index >= 0) {
                addAdjacentMatches(result, index, game -> game.getQuality() == quality);
            }
        } else {
            linearSearch(result, game -> game.getQuality() == quality);
        }
        return result;
    }
    private interface SearchCondition {
        boolean test(Game game);
    }
    private void linearSearch(ArrayList<Game> result, SearchCondition condition) {
        for (Game game : data) {
            if (condition.test(game)) {
                result.add(game);
            }
        }
    }
    private void addAdjacentMatches(ArrayList<Game> result, int index, SearchCondition condition) {
        result.add(data.get(index));
        for (int i = index-1; i >= 0 && condition.test(data.get(i)); i--) result.add(data.get(i));
        for (int i = index+1; i < data.size() && condition.test(data.get(i)); i++) result.add(data.get(i));
    }
    private void binarySearchRange(ArrayList<Game> result, int lowVal, int highVal, java.util.function.ToIntFunction<Game> extractor) {
        int low = 0, high = data.size() - 1;
        while (low <= high) {
            int mid = (low + high) / 2;
            int current = extractor.applyAsInt(data.get(mid));
            if (current >= lowVal && current <= highVal) {
                expandRange(result, mid, lowVal, highVal, extractor);
                break;
            } else if (current < lowVal) {
                low = mid + 1;
            } else {
                high = mid - 1;
            }
        }
    }
    private void expandRange(ArrayList<Game> result, int mid, int lowVal, int highVal, java.util.function.ToIntFunction<Game> extractor) {
        result.add(data.get(mid));
        for (int i = mid-1; i >= 0 && extractor.applyAsInt(data.get(i)) >= lowVal; i--) result.add(data.get(i));
        for (int i = mid+1; i < data.size() && extractor.applyAsInt(data.get(i)) <= highVal; i++) result.add(data.get(i));
    }
    public void sortByAlgorithm(String algorithm, String attribute) {
        Comparator<Game> comparator = getComparator(attribute);
        switch (algorithm.toLowerCase()) {
            case "bubblesort": bubbleSort(comparator); break;
            case "insertionsort": insertionSort(comparator); break;
            case "selectionsort": selectionSort(comparator); break;
            case "mergesort": mergeSort(comparator); break;
            case "quicksort": quickSortOptimized(0, data.size()-1, comparator); break;
            case "countingsort":
                if (attribute.equals("quality")) countingSortByQuality();
                else Collections.sort(data, comparator);
                break;
            default: Collections.sort(data, comparator); break;
        }
        this.sortedByAttribute = attribute;
    }
    private Comparator<Game> getComparator(String attribute) {
        switch (attribute.toLowerCase()) {
            case "category": return Game.byCategory;
            case "quality": return Game.byQuality;
            default: return Game.byPrice;
        }
    }
    private void bubbleSort(Comparator<Game> comparator) {
        for (int i = 0; i < data.size()-1; i++)
            for (int j = 0; j < data.size()-i-1; j++)
                if (comparator.compare(data.get(j), data.get(j+1)) > 0)
                    Collections.swap(data, j, j+1);
    }
    private void insertionSort(Comparator<Game> comparator) {
        insertionSort(0, data.size()-1, comparator);
    }
    private void insertionSort(int low, int high, Comparator<Game> comparator) {
        for (int i = low+1; i <= high; i++) {
            Game key = data.get(i);
            int j = i-1;
            while (j >= low && comparator.compare(data.get(j), key) > 0) {
                data.set(j+1, data.get(j));
                j--;
            }
            data.set(j+1, key);
        }
    }
    private void selectionSort(Comparator<Game> comparator) {
        for (int i = 0; i < data.size()-1; i++) {
            int minIdx = i;
            for (int j = i+1; j < data.size(); j++)
                if (comparator.compare(data.get(j), data.get(minIdx)) < 0) minIdx = j;
            if (minIdx != i) Collections.swap(data, i, minIdx);
        }
    }
    private void mergeSort(Comparator<Game> comparator) {
        mergeSort(0, data.size()-1, comparator);
    }
    private void mergeSort(int l, int r, Comparator<Game> comparator) {
        if (l < r) {
            int m = l + (r-l)/2;
            mergeSort(l, m, comparator);
            mergeSort(m+1, r, comparator);
            merge(l, m, r, comparator);
        }
    }
    private void merge(int l, int m, int r, Comparator<Game> comparator) {
        ArrayList<Game> temp = new ArrayList<>();
        int i = l, j = m+1;
        while (i <= m && j <= r) {
            if (comparator.compare(data.get(i), data.get(j)) <= 0) temp.add(data.get(i++));
            else temp.add(data.get(j++));
        }
        while (i <= m) temp.add(data.get(i++));
        while (j <= r) temp.add(data.get(j++));
        for (i = l, j = 0; i <= r; i++, j++) data.set(i, temp.get(j));
    }
    private void quickSortOptimized(int low, int high, Comparator<Game> comparator) {
        if (high - low < 20) {
            insertionSort(low, high, comparator);
            return;
        }
        int pivotIndex = low + random.nextInt(high - low + 1);
        Collections.swap(data, pivotIndex, high);
        int pi = partition(low, high, comparator);
        if (pi > low) quickSortOptimized(low, pi-1, comparator);
        if (low < 0 || high >= data.size() || low >= high) return;
    }
    private int partition(int low, int high, Comparator<Game> comparator) {
        Game pivot = data.get(high);
        int i = low-1;
        for (int j = low; j < high; j++) {
            if (comparator.compare(data.get(j), pivot) < 0) {
                i++;
                Collections.swap(data, i, j);
            }
        }
        if (i+1 != high) Collections.swap(data, i+1, high);
        return i+1;
    }
    public void countingSortByQuality() {
        int[] count = new int[101];
        ArrayList<Game> output = new ArrayList<>(Collections.nCopies(data.size(), null));
        for (Game game : data) count[game.getQuality()]++;
        for (int i = 1; i <= 100; i++) count[i] += count[i-1];
        for (int i = data.size()-1; i >= 0; i--) output.set(--count[data.get(i).getQuality()], data.get(i));
        for (int i = 0; i < data.size(); i++) data.set(i, output.get(i));
        this.sortedByAttribute = "quality";
    }
}
class GenerateData {
    private static final String[] GAME_WORDS = {"Dragon","Empire","Quest","Galaxy","Legends","Warrior","Shadow","Kingdom","Chronicles","Adventure","Hero","Dark"};
    private static final String[] CATEGORIES = {"Action","Adventure","Strategy","RPG","Sports","Simulation"};
    private static final Random random = new Random();
    public static ArrayList<Game> generateRandomGames(int n) {
        ArrayList<Game> games = new ArrayList<>();
        for (int i = 0; i < n; i++)
            games.add(new Game(
                    GAME_WORDS[random.nextInt(GAME_WORDS.length)] + GAME_WORDS[random.nextInt(GAME_WORDS.length)],
                    CATEGORIES[random.nextInt(CATEGORIES.length)],
                    random.nextInt(70001),
                    random.nextInt(101)
            ));
        return games;
    }
    public static void saveToFile(ArrayList<Game> games, String filename) throws IOException {
        FileWriter writer = new FileWriter(filename);
        for (Game game : games) writer.write(game.getName()+","+game.getCategory()+","+game.getPrice()+","+game.getQuality()+"\n");
        writer.close();
    }
}
class Benchmark {
    public static void main(String[] args) throws IOException {
        ArrayList<Game> small = GenerateData.generateRandomGames(100);
        ArrayList<Game> medium = GenerateData.generateRandomGames(10000);
        ArrayList<Game> large = GenerateData.generateRandomGames(1000000);
        GenerateData.saveToFile(small, "games_100.csv");
        GenerateData.saveToFile(medium, "games_10000.csv");
        GenerateData.saveToFile(large, "games_1000000.csv");
        printSortingBenchmarks(small, medium, large);
        printSearchBenchmarks(large);
    }
    private static void printSortingBenchmarks(ArrayList<Game> s, ArrayList<Game> m, ArrayList<Game> l) {
        String[] attributes = {"price","category","quality"};
        String[][] algorithms = {
                {"bubbleSort","insertionSort","selectionSort"},
                {"mergeSort","quickSort","collectionsSort","countingSort"}
        };
        System.out.println("\n=== BENCHMARK DE ORDENAMIENTO ===");
        System.out.println("| Atributo  | Algoritmo       | 10^2 (ms) | 10^4 (ms) | 10^6 (ms) |");
        System.out.println("|-----------|-----------------|-----------|-----------|-----------|");
        for (String attr : attributes) {
            for (String[] algoGroup : algorithms) {
                for (String algo : algoGroup) {
                    if (attr.equals("quality") && algo.equals("countingSort")) {
                        printSortingRow(attr, algo, s, m, l, true);
                    } else if (!algo.equals("countingSort")) {
                        printSortingRow(attr, algo, s, m, l, false);
                    }
                }
            }
        }
    }
    private static void printSortingRow(String attr, String algo, ArrayList<Game> s, ArrayList<Game> m, ArrayList<Game> l, boolean isCounting) {
        System.out.printf("| %-9s | %-15s |", attr, algo);
        Dataset smallDS = new Dataset(new ArrayList<>(s));
        System.out.printf(" %-9d |", measureSort(smallDS, algo, attr));
        Dataset mediumDS = new Dataset(new ArrayList<>(m));
        System.out.printf(" %-9d |", measureSort(mediumDS, algo, attr));
        if (algo.equals("bubbleSort") || algo.equals("insertionSort") || algo.equals("selectionSort")) {
            System.out.printf(" %-9s |%n", "N/A");
        } else {
            Dataset largeDS = new Dataset(new ArrayList<>(l));
            System.out.printf(" %-9d |%n", isCounting ?
                    measureCountingSort(largeDS) : measureSort(largeDS, algo, attr));
        }
    }
    private static long measureSort(Dataset d, String algo, String attr) {
        long start = System.currentTimeMillis();
        d.sortByAlgorithm(algo, attr);
        return System.currentTimeMillis() - start;
    }
    private static long measureCountingSort(Dataset d) {
        long start = System.currentTimeMillis();
        d.countingSortByQuality();
        return System.currentTimeMillis() - start;
    }
    private static void printSearchBenchmarks(ArrayList<Game> data) {
        Dataset unordered = new Dataset(new ArrayList<>(data));
        Dataset byPrice = new Dataset(new ArrayList<>(data)); byPrice.sortByAlgorithm("mergeSort", "price");
        Dataset byCategory = new Dataset(new ArrayList<>(data)); byCategory.sortByAlgorithm("mergeSort", "category");
        Dataset byQuality = new Dataset(new ArrayList<>(data)); byQuality.sortByAlgorithm("mergeSort", "quality");
        System.out.println("\n=== BENCHMARK DE BÚSQUEDA (10^6 elementos) ===");
        System.out.println("| Método               | Búsqueda Lineal (ms) | Búsqueda Binaria (ms) |");
        System.out.println("|----------------------|-----------------------|-----------------------|");
        printSearchRow("Por Precio", unordered, byPrice, "getGamesByPrice", 5000);
        printSearchRow("Por Rango Precio", unordered, byPrice, "getGamesByPriceRange", 3000, 7000);
        printSearchRow("Por Categoría", unordered, byCategory, "getGamesByCategory", "Action");
        printSearchRow("Por Calidad", unordered, byQuality, "getGamesByQuality", 80);
    }
    private static void printSearchRow(String label, Dataset unordered, Dataset ordered, String method, Object... params) {
        System.out.printf("| %-20s |", label);
        System.out.printf(" %-21d |", measureSearch(unordered, method, params));
        System.out.printf(" %-21d |%n", measureSearch(ordered, method, params));
    }
    private static long measureSearch(Dataset d, String method, Object... params) {
        long start = System.nanoTime();
        switch (method) {
            case "getGamesByPrice": d.getGamesByPrice((Integer)params[0]); break;
            case "getGamesByPriceRange": d.getGamesByPriceRange((Integer)params[0],(Integer)params[1]); break;
            case "getGamesByCategory": d.getGamesByCategory((String)params[0]); break;
            case "getGamesByQuality": d.getGamesByQuality((Integer)params[0]); break;
        }
        return (System.nanoTime()-start)/1000000;
    }
}
public class Main {
    public static void main(String[] args) throws IOException {
        Benchmark.main(args);
    }
}
