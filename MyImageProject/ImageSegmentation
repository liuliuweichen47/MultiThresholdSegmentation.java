import javax.imageio.ImageIO;
import java.awt.image.BufferedImage;
import java.io.File;
import java.io.IOException;

public class MultiThresholdSegmentation {

    public static void main(String[] args) {
        try {
            // 1. 讀取影像 [cite: 37]
            File inputFile = new File("input.jpg"); 
            BufferedImage originalImage = ImageIO.read(inputFile);
            int width = originalImage.getWidth();
            int height = originalImage.getHeight();

            // 2. 統計灰階直方圖 [cite: 20, 21, 27]
            int[] histogram = new int[256];
            for (int y = 0; y < height; y++) {
                for (int x = 0; x < width; x++) {
                    int rgb = originalImage.getRGB(x, y);
                    int r = (rgb >> 16) & 0xFF;
                    int g = (rgb >> 8) & 0xFF;
                    int b = rgb & 0xFF;
                    
                    // 使用加權公式轉換為灰階
                    int gray = (int)(0.299 * r + 0.587 * g + 0.114 * b);
                    histogram[gray]++;
                }
            }

            // 3. 尋找最佳門檻值 (組內變異量最小化) [cite: 22, 31]
            int tOpt = findOptimalThreshold(histogram, width * height);
            System.out.println("根據教材算法計算出的最佳門檻值為: " + tOpt);

            // 4. 根據門檻進行分割 (Identify Foreground and Background) [cite: 39]
            BufferedImage resultImage = new BufferedImage(width, height, BufferedImage.TYPE_BYTE_BINARY);
            for (int y = 0; y < height; y++) {
                for (int x = 0; x < width; x++) {
                    int rgb = originalImage.getRGB(x, y);
                    // 取得亮度值進行判斷
                    int gray = (rgb >> 16) & 0xFF;

                    if (gray > tOpt) {
                        resultImage.setRGB(x, y, 0xFFFFFFFF); // 前景：白色 [cite: 39]
                    } else {
                        resultImage.setRGB(x, y, 0xFF000000); // 背景：黑色 [cite: 39]
                    }
                }
            }

            // 5. 儲存結果
            ImageIO.write(resultImage, "jpg", new File("output.jpg"));
            System.out.println("成功！請查看資料夾中的 output.jpg");

        } catch (IOException e) {
            System.out.println("錯誤: 找不到 input.jpg 或檔案讀取失敗。");
        }
    }

    /**
     * 實作教材中的大津演算法 (Otsu's Method)
     * 透過最小化組內變異量 (Within-group variance) 來尋找最佳門檻 [cite: 31]
     */
    private static int findOptimalThreshold(int[] histogram, int totalPixels) {
        int bestT = 0;
        double minWithinVariance = Double.MAX_VALUE;

        // 遍歷所有可能的門檻值 T
        for (int t = 1; t < 255; t++) {
            double w0 = 0, w1 = 0; // 兩群的權重
            double sum0 = 0, sum1 = 0; // 兩群的亮度總和
            
            for (int i = 0; i < 256; i++) {
                if (i <= t) {
                    w0 += histogram[i];
                    sum0 += (double) i * histogram[i];
                } else {
                    w1 += histogram[i];
                    sum1 += (double) i * histogram[i];
                }
            }

            if (w0 == 0 || w1 == 0) continue;

            double m0 = sum0 / w0; // 背景平均值
            double m1 = sum1 / w1; // 前景平均值

            double var0 = 0, var1 = 0;
            for (int i = 0; i < 256; i++) {
                if (i <= t) {
                    var0 += Math.pow(i - m0, 2) * histogram[i];
                } else {
                    var1 += Math.pow(i - m1, 2) * histogram[i];
                }
            }

            // 組內變異量公式 [cite: 31]
            double withinVariance = (var0 + var1) / totalPixels;

            if (withinVariance < minWithinVariance) {
                minWithinVariance = withinVariance;
                bestT = t;
            }
        }
        return bestT;
    }
}