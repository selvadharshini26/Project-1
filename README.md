Multi language document classification system with Java code
import com.google.common.base.Optional;
import com.optimaize.langdetect.LanguageDetector;
import com.optimaize.langdetect.LanguageDetectorBuilder;
import com.optimaize.langdetect.i18n.LdLocale;
import com.optimaize.langdetect.ngram.NgramExtractors;
import com.optimaize.langdetect.profiles.LanguageProfile;
import com.optimaize.langdetect.profiles.LanguageProfileReader;
import opennlp.tools.tokenize.SimpleTokenizer;
import weka.classifiers.bayes.NaiveBayes;
import weka.core.*;
import weka.classifiers.Evaluation;

import java.io.FileReader;
import java.util.*;

public class MultilingualClassifier {

    private LanguageDetector languageDetector;
    private SimpleTokenizer tokenizer;

    public MultilingualClassifier() throws Exception {
        List<LanguageProfile> languageProfiles = new LanguageProfileReader().readAllBuiltIn();
        languageDetector = LanguageDetectorBuilder.create(NgramExtractors.standard())
            .withProfiles(languageProfiles)
            .build();
        tokenizer = SimpleTokenizer.INSTANCE;
    }

    public String detectLanguage(String text) {
        Optional<LdLocale> lang = languageDetector.detect(text);
        return lang.isPresent() ? lang.get().getLanguage() : "unknown";
    }

    public String[] tokenize(String text) {
        return tokenizer.tokenize(text);
    }

    public void classify(List<String> texts, List<String> labels) throws Exception {
        ArrayList<Attribute> attributes = new ArrayList<>();
        attributes.add(new Attribute("text", (List<String>) null));
        List<String> classValues = new ArrayList<>(new HashSet<>(labels));
        attributes.add(new Attribute("class", classValues));

        Instances dataset = new Instances("MultilingualDataset", attributes, texts.size());
        dataset.setClassIndex(1);

        for (int i = 0; i < texts.size(); i++) {
            DenseInstance instance = new DenseInstance(2);
            instance.setValue(attributes.get(0), texts.get(i));
            instance.setValue(attributes.get(1), labels.get(i));
            dataset.add(instance);
        }

        NaiveBayes classifier = new NaiveBayes();
        classifier.buildClassifier(dataset);

        Evaluation eval = new Evaluation(dataset);
        eval.crossValidateModel(classifier, dataset, 5, new Random(1));
        System.out.println(eval.toSummaryString());
    }

    public static void main(String[] args) throws Exception {
        MultilingualClassifier system = new MultilingualClassifier();

        List<String> texts = Arrays.asList(
                "Le gouvernement français a annoncé une nouvelle réforme.",
                "The president signed the new bill into law.",
                "El clima está cambiando rápidamente en la región."
        );

        List<String> labels = Arrays.asList("politics", "politics", "environment");

        for (String text : texts) {
            System.out.println("Lang: " + system.detectLanguage(text));
            System.out.println("Tokens: " + Arrays.toString(system.tokenize(text)));
        }

        system.classify(texts, labels);
    }
}
