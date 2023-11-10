# INFOTRIXS
package CurrencyConverter;
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.net.HttpURLConnection;
import java.net.URL;
import org.json.simple.JSONObject;
import org.json.simple.JSONValue;


public class CurrencyConverter {
    @SuppressWarnings("unchecked")
	public static void main(String[] args) {
        BufferedReader reader = new BufferedReader(new InputStreamReader(System.in));
        
        String baseCurrency = "USD"; 
        JSONObject favoriteCurrencies = new JSONObject();

        while (true) {
            System.out.println("\nOptions:");
            System.out.println("1. Add a favorite currency");
            System.out.println("2. View favorite currencies");
            System.out.println("3. Update exchange rates");
            System.out.println("4. Convert currency");
            System.out.println("5. Exit");
            System.out.print("Enter your choice (1 or 2 or 3 or 4 or 5): ");
            
            try {
                int choice = Integer.parseInt(reader.readLine());

                switch (choice) {
                    case 1:
                        System.out.print("Enter a currency code to add to your favorites: ");
                        String currencyCode = reader.readLine().toUpperCase();
                        favoriteCurrencies.put(currencyCode, null); 
                        System.out.println(currencyCode + " added to your favorites.");
                        break;

                    case 2:
                        System.out.println("Your favorite currencies: " + favoriteCurrencies.keySet());
                        break;

                    case 3:
                        System.out.println("Updating exchange rates...");
                        updateExchangeRates(baseCurrency, favoriteCurrencies);
                        System.out.println("Exchange rates updated.");
                        break;

                    case 4:
                        System.out.print("Enter the source currency code: ");
                        String sourceCurrency = reader.readLine().toUpperCase();
                        
                        if (favoriteCurrencies.containsKey(sourceCurrency)) {
                            System.out.print("Enter the amount to convert: ");
                            double amount = Double.parseDouble(reader.readLine());
                            double exchangeRate = getExchangeRate(baseCurrency, sourceCurrency);
                            
                            if (exchangeRate > 0) {
                                double convertedAmount = amount * exchangeRate;
                                System.out.println(amount + " " + sourceCurrency + " is approximately " + convertedAmount + " " + baseCurrency);
                            } else {
                                System.out.println("Exchange rate data not available.");
                            }
                        } else {
                            System.out.println("Source currency not found in your favorites.");
                        }
                        break;

                    case 5:
                        System.out.println("Exiting the currency converter.");
                        System.exit(0);

                    default:
                        System.out.println("Invalid choice. Please select a valid option.");
                }
            } catch (IOException | NumberFormatException e) {
                e.printStackTrace();
            }
        }
    }

    private static double getExchangeRate(String baseCurrency, String targetCurrency) {
        try {
            URL url = new URL("https://api.apilayer.com/exchangerates/data/latest?base=" + baseCurrency + "&symbols=" + targetCurrency);
            HttpURLConnection conn = (HttpURLConnection) url.openConnection();
            conn.setRequestMethod("GET");
            conn.setRequestProperty("User-Agent", "Mozilla/5.0");

            BufferedReader in = new BufferedReader(new InputStreamReader(conn.getInputStream()));
            String inputLine;
            StringBuilder response = new StringBuilder();

            while ((inputLine = in.readLine()) != null) {
                response.append(inputLine);
            }

            in.close();
            conn.disconnect();

            JSONObject jsonResponse = (JSONObject) JSONValue.parse(response.toString());

            if (jsonResponse != null && jsonResponse.containsKey("rates")) {
                JSONObject rates = (JSONObject) jsonResponse.get("rates");

                if (rates.containsKey(targetCurrency)) {
                    return Double.parseDouble(rates.get(targetCurrency).toString());
                }
            }
        } catch (IOException e) {
            e.printStackTrace();
        }

        return -1; // Default to an invalid exchange rate
    }

    @SuppressWarnings("unchecked")
	private static void updateExchangeRates(String baseCurrency, JSONObject favoriteCurrencies) {
        try {
            URL url = new URL("https://api.apilayer.com/exchangerates/data/latest?base=" + baseCurrency);
            HttpURLConnection conn = (HttpURLConnection) url.openConnection();
            conn.setRequestMethod("GET");
            conn.setRequestProperty("User-Agent", "Mozilla/5.0");

            BufferedReader in = new BufferedReader(new InputStreamReader(conn.getInputStream()));
            String inputLine;
            StringBuilder response = new StringBuilder();

            while ((inputLine = in.readLine()) != null) {
                response.append(inputLine);
            }

            in.close();
            conn.disconnect();

            JSONObject jsonResponse = (JSONObject) JSONValue.parse(response.toString());

            if (jsonResponse != null && jsonResponse.containsKey("rates")) {
                JSONObject rates = (JSONObject) jsonResponse.get("rates");

                for (Object currency : favoriteCurrencies.keySet()) {
                    String currencyCode = currency.toString();

                    if (rates.containsKey(currencyCode)) {
                        favoriteCurrencies.put(currencyCode, Double.parseDouble(rates.get(currencyCode).toString()));
                    }
                }
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

}
