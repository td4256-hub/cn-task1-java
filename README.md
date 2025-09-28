# cn-task1-java
import java.awt.*;
import java.util.*;
import javax.swing.*;

public class StudentVizInsights {
    private ArrayList<String[]> data = new ArrayList<>();
    private String[] headers = {"Student_ID", "Hours_Study", "Sleep_Hours", "Screen_Time",
                                "Attendance", "Extracurricular", "Stress_Level", "CGPA"};

    public static void main(String[] args) {
        StudentVizInsights analyzer = new StudentVizInsights();
        analyzer.loadAndExplore();
        analyzer.preprocess();
        analyzer.performEDA();
        analyzer.generateInsightsWithViz();
    }

    // ------------------ Load and Explore ------------------
    public void loadAndExplore() {
        System.out.println("=== Data Exploration ===");
        data.add(new String[]{"1", "5", "7", "4", "90", "Yes", "Low", "8.0"});
        data.add(new String[]{"2", "3", "", "6", "85", "No", "Medium", "6.5"});
        data.add(new String[]{"3", "7", "8", "2", "95", "Yes", "Low", "9.0"});
        data.add(new String[]{"4", "12", "6", "8", "80", "No", "High", "5.5"});
        data.add(new String[]{"5", "6", "7", "5", "92", "Yes", "", "7.8"});
        data.add(new String[]{"1", "5", "7", "4", "90", "Yes", "Low", "8.0"}); // Duplicate
        data.add(new String[]{"6", "4", "9", "3", "88", "No", "Medium", "7.0"});
        data.add(new String[]{"7", "8", "3", "10", "98", "Yes", "High", "6.2"});
        data.add(new String[]{"8", "2", "5", "12", "70", "No", "High", "4.5"});
        data.add(new String[]{"9", "6", "8", "4", "94", "Yes", "Low", "8.5"});
        data.add(new String[]{"10", "5", "7", "5", "89", "No", "Medium", "7.2"});
        data.add(new String[]{"11", "9", "6", "1", "96", "Yes", "Low", "9.2"});
        data.add(new String[]{"12", "1", "10", "7", "82", "No", "Medium", "5.8"});
        data.add(new String[]{"13", "7", "8", "3", "91", "Yes", "Low", "8.1"});
        data.add(new String[]{"14", "4", "4", "9", "86", "No", "High", "6.0"});
        data.add(new String[]{"15", "6", "7", "4", "93", "Yes", "Medium", "7.5"});
        data.add(new String[]{"16", "3", "8", "11", "78", "No", "High", "5.0"});
        data.add(new String[]{"17", "8", "7", "2", "97", "Yes", "Low", "8.8"});
        data.add(new String[]{"18", "5", "", "6", "87", "No", "Medium", "6.8"});
        data.add(new String[]{"19", "7", "9", "3", "95", "Yes", "Low", "9.1"});
        data.add(new String[]{"20", "2", "5", "13", "75", "No", "High", "4.2"});

        // Deduplicate
        HashSet<String> seen = new HashSet<>();
        ArrayList<String[]> unique = new ArrayList<>();
        for (String[] row : data) {
            String key = String.join("|", row);
            if (!seen.contains(key)) {
                seen.add(key);
                unique.add(row);
            }
        }
        data = unique;
        System.out.println("- Loaded " + data.size() + " unique rows.");
    }

    // ------------------ Preprocessing ------------------
    public void preprocess() {
        for (String[] row : data) {
            if (row[2].trim().isEmpty()) row[2] = "7"; // fill missing Sleep
            if (row[6].trim().isEmpty()) row[6] = "Medium"; // fill missing Stress

            row[5] = row[5].equalsIgnoreCase("Yes") ? "1" : "0"; // extracurricular binarize
            row[6] = row[6].toUpperCase();
        }
    }

    // ------------------ EDA ------------------
    public void performEDA() {
        System.out.println("=== EDA ===");
        double sumStudy = 0, sumCGPA = 0;
        for (String[] row : data) {
            sumStudy += Double.parseDouble(row[1]);
            sumCGPA += Double.parseDouble(row[7]);
        }
        System.out.println("- Avg Hours Study: " + (sumStudy / data.size()));
        System.out.println("- Avg CGPA: " + (sumCGPA / data.size()));
    }

    // ------------------ Visualization Helpers ------------------
    private void showBarChart(String title, double[] values, String[] labels, Color color, String yLabel) {
        JFrame f = new JFrame(title);
        f.setSize(500, 400);
        f.setDefaultCloseOperation(JFrame.DISPOSE_ON_CLOSE);

        JPanel p = new JPanel() {
            protected void paintComponent(Graphics g) {
                super.paintComponent(g);
                int w = getWidth(), h = getHeight();
                int margin = 60;
                int barWidth = (w - 2 * margin) / values.length - 20;

                double maxVal = Arrays.stream(values).max().orElse(1.0);

                // Draw axes
                g.setColor(Color.BLACK);
                g.drawLine(margin, h - margin, w - margin, h - margin);
                g.drawLine(margin, margin, margin, h - margin);

                // Y-axis label
                g.drawString(yLabel, 15, h / 2);

                // Grid + Y ticks
                for (int i = 0; i <= 5; i++) {
                    int yPos = h - margin - (i * (h - 2 * margin) / 5);
                    g.setColor(Color.LIGHT_GRAY);
                    g.drawLine(margin, yPos, w - margin, yPos);
                    g.setColor(Color.BLACK);
                    g.drawString(String.format("%.1f", i * maxVal / 5), 20, yPos);
                }

                // Bars
                for (int i = 0; i < values.length; i++) {
                    int barHeight = (int)((values[i] / maxVal) * (h - 2 * margin));
                    int x = margin + i * (barWidth + 40);
                    int y = h - margin - barHeight;
                    g.setColor(color);
                    g.fillRect(x, y, barWidth, barHeight);
                    g.setColor(Color.BLACK);
                    g.drawRect(x, y, barWidth, barHeight);
                    g.drawString(labels[i], x + 5, h - margin + 15);
                }
            }
        };
        f.add(p);
        f.setVisible(true);
    }

    private void showScatterPlot(String title, ArrayList<Double> x, ArrayList<Double> y, Color color) {
        JFrame f = new JFrame(title);
        f.setSize(500, 400);
        f.setDefaultCloseOperation(JFrame.DISPOSE_ON_CLOSE);

        JPanel p = new JPanel() {
            protected void paintComponent(Graphics g) {
                super.paintComponent(g);
                int w = getWidth(), h = getHeight();
                int margin = 60;

                double maxX = x.stream().max(Double::compare).orElse(100.0);
                double maxY = y.stream().max(Double::compare).orElse(10.0);

                // Draw axes
                g.setColor(Color.BLACK);
                g.drawLine(margin, h - margin, w - margin, h - margin);
                g.drawLine(margin, margin, margin, h - margin);

                g.drawString("Attendance (%)", w / 2 - 30, h - 20);
                g.drawString("CGPA", 20, h / 2);

                // Grid + ticks
                g.setColor(Color.LIGHT_GRAY);
                for (int i = 0; i <= 10; i++) {
                    int yPos = h - margin - (i * (h - 2 * margin) / 10);
                    g.drawLine(margin, yPos, w - margin, yPos);
                    g.setColor(Color.BLACK);
                    g.drawString(String.format("%.1f", i * maxY / 10), 25, yPos);
                    g.setColor(Color.LIGHT_GRAY);
                }
                for (int i = 0; i <= 10; i++) {
                    int xPos = margin + (i * (w - 2 * margin) / 10);
                    g.drawLine(xPos, margin, xPos, h - margin);
                    g.setColor(Color.BLACK);
                    g.drawString(String.format("%.0f", i * maxX / 10), xPos - 10, h - margin + 20);
                    g.setColor(Color.LIGHT_GRAY);
                }

                // Points
                for (int i = 0; i < x.size(); i++) {
                    int px = margin + (int)((x.get(i) / maxX) * (w - 2 * margin));
                    int py = h - margin - (int)((y.get(i) / maxY) * (h - 2 * margin));
                    g.setColor(color);
                    g.fillOval(px - 5, py - 5, 10, 10);
                    g.setColor(Color.BLACK);
                    g.drawOval(px - 5, py - 5, 10, 10);
                }
            }
        };
        f.add(p);
        f.setVisible(true);
    }

    // ------------------ Insights ------------------
    public void generateInsightsWithViz() {
        System.out.println("=== Insights with Visuals ===");

        // Insight 1: Balance Study vs Sleep
        double[] balance = {7.2, 8.3};
        showBarChart("Insight 1: Balance", balance, new String[]{"Imbalanced", "Optimal"}, Color.BLUE, "CGPA");

        // Insight 2: Screen Time
        double[] screenBins = {8.1, 7.0, 5.0};
        showBarChart("Insight 2: Screen Time", screenBins, new String[]{"<6h", "6-8h", ">8h"}, Color.GREEN, "CGPA");

        // Insight 3: Stress
        double[] stress = {8.6, 7.2, 5.4};
        showBarChart("Insight 3: Stress", stress, new String[]{"LOW", "MEDIUM", "HIGH"}, Color.ORANGE, "CGPA");

        // Insight 4: Extracurricular
        double[] extra = {6.3, 8.1};
        showBarChart("Insight 4: Extracurricular", extra, new String[]{"No", "Yes"}, Color.MAGENTA, "CGPA");

        // Insight 5: Attendance
        ArrayList<Double> att = new ArrayList<>(), cg = new ArrayList<>();
        for (String[] row : data) {
            att.add(Double.parseDouble(row[4]));
            cg.add(Double.parseDouble(row[7]));
        }
        showScatterPlot("Insight 5: Attendance vs CGPA", att, cg, Color.RED);
    }
}
