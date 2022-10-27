# gsutil
package test;
import java.io.BufferedWriter;
import java.io.File;
import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;
import java.util.Arrays;
import java.util.Comparator;
import java.util.Date;

public class GSCLITest
{
    public static void main(String[] args) throws Exception
    {
        System.out.println();
        System.out.println("************* Arguments *****************");
        System.out.println("1. Path to create folder on local -> " + args[0]);
        System.out.println("2. Generate Data -> " + args[1]);
        System.out.println("3. No. of files to be created for each size -> " + args[2]);
        System.out.println("4. Path of google cloud storage to be copied on -> " + args[3]);
        System.out.println("5. Local path at which files to be copied from cloud storage -> " + args[4]);
        System.out.println("*******************************************");
        System.out.println();

        String folderPath = args[0] + File.separator + "metadata";
        boolean generateData = Boolean.parseBoolean(args[1]);
        int fileCount = Integer.parseInt(args[2]);
        String gfsPath = args[3];
        String pathOfLocal = args[4];

        if (generateData)
        {
            File f1 = new File(folderPath);
            if (f1.exists())
            {
                System.out.println("Deleting folder on local at path " + folderPath);
                Files.walk(Paths.get(folderPath))
                        .sorted(Comparator.reverseOrder())
                        .map(Path::toFile)
                        .forEach(File::delete);
            }

            generateData(folderPath, fileCount);
        }

        long startTime = System.currentTimeMillis();
        copyData(folderPath, gfsPath);
        System.out.println("*******************************************");
        System.out.println("Copy completed from local to cloud storage. TimeTaken: " + (System.currentTimeMillis() - startTime) + "ms");
        System.out.println("*******************************************");

        startTime = System.currentTimeMillis();
        File localFile = new File(pathOfLocal);
        if (!localFile.exists()) {
            localFile.mkdirs();
        }
        copyData(gfsPath, pathOfLocal);
        System.out.println("*******************************************");
        System.out.println("Copy completed from cloud storage to local. TimeTaken: " + (System.currentTimeMillis() - startTime) + "ms");
        System.out.println("*******************************************");
    }

    private static void generateData(String folderPath, int no) throws IOException
    {
        createDirectoryOnLocal(folderPath);
        long startTime = System.currentTimeMillis();
        String fiveMBFilePath = folderPath + File.separator + getClientGuid("");
        String fiftyMBFilePath = folderPath + File.separator + getClientGuid("");
        String fiveHundredMBFilePath = folderPath + File.separator + getClientGuid("");
        String twoGBFilePath = folderPath + File.separator + getClientGuid("");
        for (int i = 0; i < no; i++)
        {
            generateFile(fiveMBFilePath, 5L * 1024L * 1024L);
            generateFile(fiftyMBFilePath, 50L * 1024L * 1024L);
            generateFile(fiveHundredMBFilePath, 500L * 1024L * 1024L);
            generateFile(twoGBFilePath, 2L * 1024L * 1024L * 1024L);
        }
        System.out.println("Files created successfully. Total time taken: " + (System.currentTimeMillis() - startTime) + " ms");
    }

    private static void generateFile(String absolutePath, long fileSize) throws IOException {

        String fileName = getClientGuid("");
        String folderName = absolutePath + File.separator + fileName;
        createDirectoryOnLocal(folderName);
        String fileNameWithPath = folderName + File.separator + fileName;
        System.out.println("Creating file " + fileNameWithPath);
        Path fullPath = new File(fileNameWithPath).toPath();
        long lines = fileSize / 160L;
        System.out.println("No. of lines to be written " + lines);
        System.out.println();
        try (BufferedWriter bw = Files.newBufferedWriter(fullPath)) {
            for (int i = 0; i < lines; i++) {
                String line = getClientGuid("") + getClientGuid("")
                        + getClientGuid("") + getClientGuid("")
                        + getClientGuid("") + getClientGuid("")
                        + getClientGuid("") + getClientGuid("")
                        + getClientGuid("") + getClientGuid("");
                bw.write(line);
            }
        }
    }

    public static void createDirectoryOnLocal(String folderPath) {
        File f1 = new File(folderPath);
        boolean bool2 = f1.mkdirs();
        if (bool2) {
            System.out.println("Folder is created successfully at " + folderPath);
        } else {
            System.out.println("Error Found!");
        }
    }

    public static String getClientGuid(String ip) {
        Date date = new Date();

        String guid = "" + date.getTime();
        guid += getUniqueIdentifier(1);

        // In case of ipv6, colons will be present in IP, so replacing colon
        // with dots
        ip = ip.replaceAll(":", ".");

        while (ip.indexOf(".") != -1) {
            String temp = ip.substring(0, ip.indexOf("."));
            if (temp.length() == 1) {
                temp += getUniqueIdentifier(2);
            } else if (temp.length() == 2) {
                temp += getUniqueIdentifier(1);
            }
            ip = ip.substring(ip.indexOf(".") + 1);
            guid += temp;
        }
        if (ip.length() == 1) {
            ip += getUniqueIdentifier(2);
        } else if (ip.length() == 2) {
            ip += getUniqueIdentifier(1);
        }
        guid += ip;
        guid += getUniqueIdentifier(1);
        guid += getUniqueIdentifier(5);
        return guid;
    }

    public static long getUniqueIdentifier(int digitCount) {
        int m = 1;
        for (int i = 0; i < digitCount; i++) {
            m = m * 10;
        }
        long rNo = Math.round(Math.random() * m);
        String temRNo = "" + rNo;
        if (temRNo.length() != digitCount)
            return getUniqueIdentifier(digitCount);
        return rNo;
    }

    private static void copyData(String sourceFolderPath, String destFolderPath) {
        try {
            executeCommand(sourceFolderPath, destFolderPath);
        } catch (Exception ex) {
            System.out.println(ex.getMessage());
        }
    }

    public static boolean executeCommand(String sourceFolderPath, String destFolderPath) throws Exception {
        String command = "gsutil -m cp -r " + sourceFolderPath + " " + destFolderPath;
        String[] cliCommands = new String[]{"bash", "-c", command};
        Process p = null;
        System.out.println("Arguments array: " + Arrays.toString(cliCommands));
        try {
            p = Runtime.getRuntime().exec(cliCommands);
            boolean success = p.waitFor() == 0;
            System.out.println("exit value: " + p.exitValue() + " success: " + success);
            return p.exitValue() == 0;
        } catch (Exception e) {
            System.out.println(e);
            return false;
        } finally {
            try {
                if (null != p) {
                    p.destroy();
                }
            } catch (Exception e) {
                System.out.println(e);
            }
        }
    }
}
