
import React, { useState, useMemo, useContext, useRef, useEffect, useCallback, createContext } from 'react';
import ReactDOM from 'react-dom/client';
import { GoogleGenAI, GenerateContentResponse, Type } from "@google/genai";

// --- TYPES ---
export type ToolId =
    | 'dashboard'
    | 'image-to-text'
    | 'ai-image-generator'
    | 'image-compressor'
    | 'image-editor'
    | 'background-remover'
    | 'ai-video-generator'
    | 'video-editor'
    | 'youtube-thumbnail'
    | 'youtube-description-tags'
    | 'youtube-script-generator'
    | 'word-to-voice'
    | 'background-music'
    | 'qr-code-design'
    | 'media-to-qr'
    | 'qr-code-generator'
    | 'pdf-editor';

export interface Tool {
    id: ToolId;
    name: { en: string; hi: string };
    description: { en: string; hi: string };
    category: 'image' | 'video' | 'youtube' | 'audio' | 'qr' | 'document';
    icon: React.ComponentType<{ className?: string }>;
    isImplemented: boolean;
}

// --- CONTEXT ---
export type Language = 'en' | 'hi';
export interface LanguageContextType {
    language: Language;
    setLanguage: (language: Language) => void;
}
export const LanguageContext = createContext<LanguageContextType>({
    language: 'en',
    setLanguage: () => {},
});


// --- ICONS ---
const LogoIcon: React.FC<{ className?: string }> = ({ className }) => (
    <svg className={className} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round">
        <path d="M12 2L2 7l10 5 10-5-10-5zM2 17l10 5 10-5M2 12l10 5 10-5"></path>
    </svg>
);
const MailIcon: React.FC<{ className?: string }> = ({ className }) => (
    <svg className={className} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round">
        <path d="M4 4h16c1.1 0 2 .9 2 2v12c0 1.1-.9 2-2 2H4c-1.1 0-2-.9-2-2V6c0-1.1.9-2 2-2z"></path><polyline points="22,6 12,13 2,6"></polyline>
    </svg>
);
const PhoneIcon: React.FC<{ className?: string }> = ({ className }) => (
    <svg className={className} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round">
        <path d="M22 16.92v3a2 2 0 0 1-2.18 2 19.79 19.79 0 0 1-8.63-3.07 19.5 19.5 0 0 1-6-6 19.79 19.79 0 0 1-3.07-8.67A2 2 0 0 1 4.11 2h3a2 2 0 0 1 2 1.72 12.84 12.84 0 0 0 .7 2.81 2 2 0 0 1-.45 2.11L8.09 9.91a16 16 0 0 0 6 6l1.27-1.27a2 2 0 0 1 2.11-.45 12.84 12.84 0 0 0 2.81.7A2 2 0 0 1 22 16.92z"></path>
    </svg>
);
const HomeIcon: React.FC<{ className?: string }> = ({ className }) => (
    <svg className={className} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round">
        <path d="M3 9l9-7 9 7v11a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2z"></path><polyline points="9 22 9 12 15 12 15 22"></polyline>
    </svg>
);
const TextIcon: React.FC<{ className?: string }> = ({ className }) => (
    <svg className={className} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round">
        <path d="M17 6.1H3" /><path d="M21 12.1H3" /><path d="M15.1 18.1H3" />
    </svg>
);
const ImageIcon: React.FC<{ className?: string }> = ({ className }) => (
    <svg className={className} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round">
        <rect x="3" y="3" width="18" height="18" rx="2" ry="2"></rect><circle cx="8.5" cy="8.5" r="1.5"></circle><polyline points="21 15 16 10 5 21"></polyline>
    </svg>
);
const VideoIcon: React.FC<{ className?: string }> = ({ className }) => (
    <svg className={className} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round">
        <polygon points="23 7 16 12 23 17 23 7"></polygon><rect x="1" y="5" width="15" height="14" rx="2" ry="2"></rect>
    </svg>
);
const YoutubeIcon: React.FC<{ className?: string }> = ({ className }) => (
    <svg className={className} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round">
        <path d="M22.54 6.42a2.78 2.78 0 0 0-1.94-2C18.88 4 12 4 12 4s-6.88 0-8.6.46a2.78 2.78 0 0 0-1.94 2A29 29 0 0 0 1 11.75a29 29 0 0 0 .46 5.33A2.78 2.78 0 0 0 3.4 19c1.72.46 8.6.46 8.6.46s6.88 0 8.6-.46a2.78 2.78 0 0 0 1.94-2 29 29 0 0 0 .46-5.25 29 29 0 0 0-.46-5.33z"></path><polygon points="9.75 15.02 15.5 11.75 9.75 8.48 9.75 15.02"></polygon>
    </svg>
);
const QrCodeIcon: React.FC<{ className?: string }> = ({ className }) => (
    <svg className={className} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round">
        <rect x="3" y="3" width="7" height="7"></rect><rect x="14" y="3" width="7" height="7"></rect><rect x="3" y="14" width="7" height="7"></rect><line x1="14" y1="14" x2="14.01" y2="14"></line><line x1="17" y1="14" x2="17.01" y2="14"></line><line x1="20" y1="14" x2="20.01" y2="14"></line><line x1="14" y1="17" x2="14.01" y2="17"></line><line x1="17" y1="17" x2="17.01" y2="17"></line><line x1="20" y1="17" x2="20.01" y2="17"></line><line x1="14" y1="20" x2="14.01" y2="20"></line><line x1="17" y1="20" x2="17.01" y2="20"></line><line x1="20" y1="20" x2="20.01" y2="20"></line>
    </svg>
);
const MusicIcon: React.FC<{ className?: string }> = ({ className }) => (
    <svg className={className} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round">
        <path d="M9 18V5l12-2v13"></path><circle cx="6" cy="18" r="3"></circle><circle cx="18" cy="16" r="3"></circle>
    </svg>
);
const FileTextIcon: React.FC<{ className?: string }> = ({ className }) => (
    <svg className={className} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round">
        <path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z"></path><polyline points="14 2 14 8 20 8"></polyline><line x1="16" y1="13" x2="8" y2="13"></line><line x1="16" y1="17" x2="8" y2="17"></line><polyline points="10 9 9 9 8 9"></polyline>
    </svg>
);
const ScissorsIcon: React.FC<{ className?: string }> = ({ className }) => (
    <svg className={className} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round">
        <circle cx="6" cy="6" r="3"></circle><circle cx="6" cy="18" r="3"></circle><line x1="20" y1="4" x2="8.12" y2="15.88"></line><line x1="14.47" y1="14.48" x2="20" y2="20"></line><line x1="8.12" y1="8.12" x2="12" y2="12"></line>
    </svg>
);
const CompressIcon: React.FC<{ className?: string }> = ({ className }) => (
    <svg className={className} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round">
        <path d="M14 10l-4 4m0 0l4 4m-4-4H3m11-4h7m-7 4h7M3 10h7m0 4H3"></path>
    </svg>
);
const WandIcon: React.FC<{ className?: string }> = ({ className }) => (
    <svg className={className} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round">
        <path d="M15 4V2m0 14v-2m-7.5 5.5L4 20m16-16l-3.5 3.5M2 6l3.5 3.5m0 7L2 20m12.5-10.5L18 6m-3.5 3.5L8 16"></path>
    </svg>
);
const MicIcon: React.FC<{ className?: string }> = ({ className }) => (
    <svg className={className} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round">
        <path d="M12 1a3 3 0 0 0-3 3v8a3 3 0 0 0 6 0V4a3 3 0 0 0-3-3z"></path><path d="M19 10v2a7 7 0 0 1-14 0v-2"></path><line x1="12" y1="19" x2="12" y2="23"></line><line x1="8" y1="23" x2="16" y2="23"></line>
    </svg>
);
const TagIcon: React.FC<{ className?: string }> = ({ className }) => (
    <svg className={className} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round">
        <path d="M20.59 13.41l-7.17 7.17a2 2 0 0 1-2.83 0L2 12V2h10l8.59 8.59a2 2 0 0 1 0 2.82z"></path><line x1="7" y1="7" x2="7.01" y2="7"></line>
    </svg>
);
const ScriptIcon: React.FC<{ className?: string }> = ({ className }) => (
    <svg className={className} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round">
        <path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z"></path><polyline points="14 2 14 8 20 8"></polyline><path d="M8 12h8m-8 4h4"></path>
    </svg>
);
const SearchIcon: React.FC<{ className?: string }> = ({ className }) => (
    <svg className={className} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round">
        <circle cx="11" cy="11" r="8"></circle><line x1="21" y1="21" x2="16.65" y2="16.65"></line>
    </svg>
);
const UploadCloudIcon: React.FC<{ className?: string }> = ({ className }) => (
    <svg className={className} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round">
        <path d="M18 10h-1.26A8 8 0 1 0 9 20h9a5 5 0 0 0 0-10z" /><path d="M12 14l-4-4h8l-4 4" />
    </svg>
);
const CopyIcon: React.FC<{ className?: string }> = ({ className }) => (
    <svg className={className} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round">
        <rect x="9" y="9" width="13" height="13" rx="2" ry="2"></rect><path d="M5 15H4a2 2 0 0 1-2-2V4a2 2 0 0 1 2-2h9a2 2 0 0 1 2 2v1"></path>
    </svg>
);


// --- GEMINI SERVICE ---
const API_KEY = "AIzaSyAmn1Y4qBKdie20lZqpn8OLvYF_JQ5E_nA";
const ai = new GoogleGenAI({ apiKey: API_KEY });

const fileToGenerativePart = async (file: File) => {
    const base64EncodedDataPromise = new Promise<string>((resolve) => {
        const reader = new FileReader();
        reader.onloadend = () => resolve((reader.result as string).split(',')[1]);
        reader.readAsDataURL(file);
    });
    return {
        inlineData: { data: await base64EncodedDataPromise, mimeType: file.type },
    };
};

const generateTextFromImage = async (prompt: string, image: File): Promise<string> => {
    try {
        const imagePart = await fileToGenerativePart(image);
        const textPart = { text: prompt };
        const response: GenerateContentResponse = await ai.models.generateContent({
            model: 'gemini-2.5-flash',
            contents: { parts: [imagePart, textPart] },
        });
        return response.text;
    } catch (error) {
        console.error("Error generating text from image:", error);
        throw new Error("Failed to extract text from image. Please try again.");
    }
};

const generateImage = async (prompt: string, aspectRatio: '1:1' | '16:9' | '9:16'): Promise<string> => {
    try {
        const response = await ai.models.generateImages({
            model: 'imagen-3.0-generate-002',
            prompt: prompt,
            config: {
                numberOfImages: 1,
                outputMimeType: 'image/jpeg',
                aspectRatio,
            },
        });
        const base64ImageBytes: string = response.generatedImages[0].image.imageBytes;
        return `data:image/jpeg;base64,${base64ImageBytes}`;
    } catch (error) {
        console.error("Error generating image:", error);
        throw new Error("Failed to generate image. Please try a different prompt.");
    }
};

const generateVideo = async (prompt: string) => {
    try {
        let operation = await ai.models.generateVideos({
            model: 'veo-2.0-generate-001',
            prompt: prompt,
            config: {
                numberOfVideos: 1
            }
        });
        return operation;
    } catch (error) {
        console.error("Error generating video:", error);
        throw new Error("Failed to start video generation. Please try again.");
    }
};

const checkVideoStatus = async (operation: any) => {
     try {
        return await ai.operations.getVideosOperation({ operation: operation });
     } catch (error) {
        console.error("Error checking video status:", error);
        throw new Error("Failed to check video status.");
     }
};

const generateYouTubeContent = async (topic: string, type: 'description' | 'script'): Promise<any> => {
    let prompt = '';
    let responseSchema: any = null;

    if (type === 'description') {
        prompt = `Generate a compelling YouTube video description, 5 relevant tags, and 3 relevant hashtags for a video about: "${topic}".`;
        responseSchema = {
            type: Type.OBJECT,
            properties: {
                description: { type: Type.STRING, description: 'The YouTube video description.' },
                tags: { type: Type.ARRAY, items: { type: Type.STRING }, description: 'An array of 5 relevant tags.' },
                hashtags: { type: Type.ARRAY, items: { type: Type.STRING }, description: 'An array of 3 relevant hashtags (without the #).' }
            },
        };
    } else { // script
        prompt = `Write a script for a YouTube video about "${topic}". The script should have an engaging introduction, a main body with clear points, and a concluding call to action. Format it in sections: "Intro", "Main Body", "Outro".`;
    }
    
    try {
        const response = await ai.models.generateContent({
            model: "gemini-2.5-flash",
            contents: prompt,
            config: responseSchema ? {
                responseMimeType: "application/json",
                responseSchema,
            } : {},
        });

        if (responseSchema) {
            return JSON.parse(response.text);
        }
        return response.text;

    } catch (error) {
        console.error(`Error generating YouTube ${type}:`, error);
        throw new Error(`Failed to generate YouTube ${type}. Please try again.`);
    }
};


// --- GENERIC COMPONENTS ---
interface ToolContainerProps {
    tool: Tool;
    children: React.ReactNode;
}
const ToolContainer: React.FC<ToolContainerProps> = ({ tool, children }) => {
    const { language } = useContext(LanguageContext);

    return (
        <div className="max-w-4xl mx-auto">
            <div className="mb-8">
                <div className="flex items-center space-x-4">
                    <div className="bg-blue-100 text-blue-600 rounded-lg p-3">
                        <tool.icon className="h-7 w-7" />
                    </div>
                    <div>
                        <h1 className="text-3xl font-bold text-slate-800">{tool.name[language]}</h1>
                        <p className="text-md text-slate-500">{tool.description[language]}</p>
                    </div>
                </div>
            </div>
            <div className="bg-white p-6 sm:p-8 rounded-xl border border-slate-200">
                {children}
            </div>
        </div>
    );
};


// --- TOOL COMPONENTS ---
interface ToolProps {
    tool: Tool;
}

const PlaceholderTool: React.FC<ToolProps> = ({ tool }) => {
    const { language } = useContext(LanguageContext);
    const t = i18n[language];

    return (
        <ToolContainer tool={tool}>
            <div className="text-center py-12">
                <h2 className="text-2xl font-semibold text-slate-700 mb-2">{t.toolComingSoon}</h2>
                <p className="text-slate-500">{t.toolComingSoonDesc}</p>
            </div>
        </ToolContainer>
    );
};

const ImageToTextTool: React.FC<ToolProps> = ({ tool }) => {
    const { language } = useContext(LanguageContext);
    const [imageFile, setImageFile] = useState<File | null>(null);
    const [previewUrl, setPreviewUrl] = useState<string | null>(null);
    const [extractedText, setExtractedText] = useState<string>('');
    const [isLoading, setIsLoading] = useState<boolean>(false);
    const [error, setError] = useState<string>('');
    const [copySuccess, setCopySuccess] = useState<string>('');

    const handleFileChange = (event: React.ChangeEvent<HTMLInputElement>) => {
        const file = event.target.files?.[0];
        if (file) {
            setImageFile(file);
            setPreviewUrl(URL.createObjectURL(file));
            setExtractedText('');
            setError('');
        }
    };

    const handleExtractText = useCallback(async () => {
        if (!imageFile) return;
        setIsLoading(true);
        setError('');
        setExtractedText('');
        try {
            const text = await generateTextFromImage('Extract all text from this image.', imageFile);
            setExtractedText(text);
        } catch (e: any) {
            setError(e.message || 'An unknown error occurred.');
        } finally {
            setIsLoading(false);
        }
    }, [imageFile]);

    const copyToClipboard = () => {
        navigator.clipboard.writeText(extractedText).then(() => {
            setCopySuccess('Copied!');
            setTimeout(() => setCopySuccess(''), 2000);
        }, () => {
            setCopySuccess('Failed to copy!');
            setTimeout(() => setCopySuccess(''), 2000);
        });
    };

    return (
        <ToolContainer tool={tool}>
            <div className="grid grid-cols-1 md:grid-cols-2 gap-8">
                <div>
                    <label htmlFor="file-upload" className="cursor-pointer">
                        <div className="border-2 border-dashed border-slate-300 rounded-lg p-6 text-center hover:border-blue-500 transition-colors">
                            {previewUrl ? (
                                <img src={previewUrl} alt="Preview" className="max-h-60 mx-auto rounded-md" />
                            ) : (
                                <>
                                    <UploadCloudIcon className="mx-auto h-12 w-12 text-slate-400" />
                                    <p className="mt-2 text-sm text-slate-600">
                                        {language === 'en' ? 'Click to upload or drag and drop' : 'अपलोड करने के लिए क्लिक करें या खींचें और छोड़ें'}
                                    </p>
                                    <p className="text-xs text-slate-500">{language === 'en' ? 'PNG, JPG, WEBP' : 'पीएनजी, जेपीजी, वेबपी'}</p>
                                </>
                            )}
                        </div>
                    </label>
                    <input id="file-upload" name="file-upload" type="file" className="sr-only" accept="image/png, image/jpeg, image/webp" onChange={handleFileChange} />
                    <button
                        onClick={handleExtractText}
                        disabled={!imageFile || isLoading}
                        className="w-full mt-4 bg-blue-600 text-white font-semibold py-2.5 px-4 rounded-lg hover:bg-blue-700 disabled:bg-blue-300 disabled:cursor-not-allowed transition-colors"
                    >
                        {isLoading ? (language === 'en' ? 'Extracting...' : 'निकाल रहा है...') : (language === 'en' ? 'Extract Text' : 'टेक्स्ट निकालें')}
                    </button>
                </div>
                <div>
                    <div className="relative">
                         <textarea
                            readOnly
                            value={extractedText}
                            placeholder={language === 'en' ? 'Extracted text will appear here...' : 'निकाला गया टेक्स्ट यहां दिखाई देगा...'}
                            className="w-full h-80 p-3 border border-slate-300 rounded-lg resize-none focus:ring-2 focus:ring-blue-500 focus:border-blue-500"
                        />
                        {extractedText && (
                            <button onClick={copyToClipboard} className="absolute top-2 right-2 p-2 rounded-md bg-slate-100 hover:bg-slate-200 transition">
                                {copySuccess ? <span className="text-xs text-blue-600">{copySuccess}</span> : <CopyIcon className="h-5 w-5 text-slate-500"/>}
                            </button>
                        )}
                    </div>
                   
                    {error && <p className="text-red-500 text-sm mt-2">{error}</p>}
                </div>
            </div>
        </ToolContainer>
    );
};

const ImageGeneratorTool: React.FC<ToolProps> = ({ tool }) => {
    const { language } = useContext(LanguageContext);
    const [prompt, setPrompt] = useState('');
    const [aspectRatio, setAspectRatio] = useState<'1:1' | '16:9' | '9:16'>('1:1');
    const [generatedImage, setGeneratedImage] = useState<string | null>(null);
    const [isLoading, setIsLoading] = useState<boolean>(false);
    const [error, setError] = useState<string>('');

    const handleGenerateImage = useCallback(async () => {
        if (!prompt) return;
        setIsLoading(true);
        setError('');
        setGeneratedImage(null);
        try {
            const imageUrl = await generateImage(prompt, aspectRatio);
            setGeneratedImage(imageUrl);
        } catch (e: any) {
            setError(e.message || 'An unknown error occurred.');
        } finally {
            setIsLoading(false);
        }
    }, [prompt, aspectRatio]);

    return (
        <ToolContainer tool={tool}>
            <div className="space-y-6">
                <div>
                    <label htmlFor="prompt" className="block text-sm font-medium text-slate-700 mb-1">
                        {language === 'en' ? 'Describe the image you want to create' : 'वह छवि बताएं जिसे आप बनाना चाहते हैं'}
                    </label>
                    <textarea
                        id="prompt"
                        rows={3}
                        value={prompt}
                        onChange={(e) => setPrompt(e.target.value)}
                        placeholder={language === 'en' ? 'e.g., A blue robot holding a red skateboard in a futuristic city' : 'उदा., एक भविष्य के शहर में एक लाल स्केटबोर्ड पकड़े हुए एक नीला रोबोट'}
                        className="w-full p-3 border border-slate-300 rounded-lg resize-none focus:ring-2 focus:ring-blue-500 focus:border-blue-500"
                    />
                </div>
                <div>
                    <label className="block text-sm font-medium text-slate-700 mb-2">
                        {language === 'en' ? 'Aspect Ratio' : 'आस्पेक्ट रेशियो'}
                    </label>
                    <div className="flex space-x-2">
                        {(['1:1', '16:9', '9:16'] as const).map((ratio) => (
                             <button
                                key={ratio}
                                onClick={() => setAspectRatio(ratio)}
                                className={`px-4 py-2 rounded-lg border text-sm font-semibold transition-colors ${
                                    aspectRatio === ratio
                                    ? 'bg-blue-600 text-white border-blue-600'
                                    : 'bg-white text-slate-700 border-slate-300 hover:bg-slate-50'
                                }`}
                            >
                                {ratio}
                            </button>
                        ))}
                    </div>
                </div>
                <button
                    onClick={handleGenerateImage}
                    disabled={!prompt || isLoading}
                    className="w-full flex items-center justify-center bg-blue-600 text-white font-semibold py-2.5 px-4 rounded-lg hover:bg-blue-700 disabled:bg-blue-300 disabled:cursor-not-allowed transition-colors"
                >
                    <WandIcon className="h-5 w-5 mr-2" />
                    {isLoading ? (language === 'en' ? 'Generating...' : 'बना रहा है...') : (language === 'en' ? 'Generate Image' : 'छवि बनाएं')}
                </button>
                {error && <p className="text-red-500 text-sm text-center">{error}</p>}
                <div className="mt-6">
                    {isLoading && (
                        <div className="w-full aspect-square bg-slate-100 rounded-lg flex items-center justify-center animate-pulse">
                            <ImageIcon className="h-16 w-16 text-slate-300"/>
                        </div>
                    )}
                    {generatedImage && (
                        <div>
                            <img src={generatedImage} alt="Generated art" className="w-full rounded-lg border border-slate-200" />
                            <a 
                                href={generatedImage} 
                                download={`pixelcraft-${Date.now()}.jpg`}
                                className="w-full block text-center mt-4 bg-slate-700 text-white font-semibold py-2.5 px-4 rounded-lg hover:bg-slate-800 transition-colors"
                            >
                                {language === 'en' ? 'Download Image' : 'छवि डाउनलोड करें'}
                            </a>
                        </div>
                    )}
                </div>
            </div>
        </ToolContainer>
    );
};

const LOADING_MESSAGES = {
    en: [
        "Warming up the 3D render farm...",
        "Teaching the AI about cinematography...",
        "Compositing final scenes...",
        "Adding cinematic sound effects...",
        "Polishing the pixels...",
        "This can take a few minutes, hang tight!",
    ],
    hi: [
        "3डी रेंडर फार्म को गर्म किया जा रहा है...",
        "एआई को सिनेमैटोग्राफी सिखाया जा रहा है...",
        "अंतिम दृश्यों को कंपोजिट किया जा रहा है...",
        "सिनेमैटिक साउंड इफेक्ट्स जोड़े जा रहे हैं...",
        "पिक्सल को पॉलिश किया जा रहा है...",
        "इसमें कुछ मिनट लग सकते हैं, कृपया प्रतीक्षा करें!",
    ]
};
const VideoGeneratorTool: React.FC<ToolProps> = ({ tool }) => {
    const { language } = useContext(LanguageContext);
    const t = LOADING_MESSAGES[language];
    const [prompt, setPrompt] = useState('');
    const [generatedVideoUrl, setGeneratedVideoUrl] = useState<string | null>(null);
    const [isLoading, setIsLoading] = useState<boolean>(false);
    const [isFetchingVideo, setIsFetchingVideo] = useState<boolean>(false);
    const [error, setError] = useState<string>('');
    const [loadingMessage, setLoadingMessage] = useState(t[0]);

    const operationRef = useRef<any>(null);
    const pollingIntervalRef = useRef<number | null>(null);
    const messageIntervalRef = useRef<number | null>(null);

    useEffect(() => {
        // Cleanup function to revoke the blob URL to prevent memory leaks
        return () => {
            if (generatedVideoUrl && generatedVideoUrl.startsWith('blob:')) {
                URL.revokeObjectURL(generatedVideoUrl);
            }
        };
    }, [generatedVideoUrl]);

    const stopPolling = () => {
        if (pollingIntervalRef.current) clearInterval(pollingIntervalRef.current);
        if (messageIntervalRef.current) clearInterval(messageIntervalRef.current);
        pollingIntervalRef.current = null;
        messageIntervalRef.current = null;
    };
    
    const pollOperation = useCallback(async () => {
        if (!operationRef.current) return;
        try {
            const updatedOperation = await checkVideoStatus(operationRef.current);
            operationRef.current = updatedOperation;

            if (updatedOperation.done) {
                stopPolling();
                setIsLoading(false);
                const downloadLink = updatedOperation.response?.generatedVideos?.[0]?.video?.uri;
                if (downloadLink) {
                    setIsFetchingVideo(true);
                    try {
                        const finalUrl = `${downloadLink}&key=${API_KEY}`;
                        const response = await fetch(finalUrl);
                        if (!response.ok) {
                            throw new Error(`Failed to fetch video: ${response.statusText}`);
                        }
                        const videoBlob = await response.blob();
                        const objectUrl = URL.createObjectURL(videoBlob);
                        setGeneratedVideoUrl(objectUrl);
                    } catch (fetchError: any) {
                         setError((language === 'en' ? 'Failed to download video file: ' : 'वीडियो फ़ाइल डाउनलोड करने में विफल: ') + fetchError.message);
                    } finally {
                        setIsFetchingVideo(false);
                    }
                } else {
                    setError(language === 'en' ? 'Video generation finished but no video was found.' : 'वीडियो जनरेशन समाप्त हो गया लेकिन कोई वीडियो नहीं मिला।');
                }
            }
        } catch (e: any) {
            setError(e.message || 'An unknown error occurred during polling.');
            setIsLoading(false);
            stopPolling();
        }
    }, [language]);
    
    const handleGenerateVideo = useCallback(async () => {
        if (!prompt) return;
        setIsLoading(true);
        setError('');
        setGeneratedVideoUrl(null);
        setLoadingMessage(t[0]);
        let msgIndex = 0;
        messageIntervalRef.current = window.setInterval(() => {
            msgIndex = (msgIndex + 1) % t.length;
            setLoadingMessage(t[msgIndex]);
        }, 5000);

        try {
            const initialOperation = await generateVideo(prompt);
            operationRef.current = initialOperation;
            pollingIntervalRef.current = window.setInterval(pollOperation, 10000);
        } catch (e: any) {
            setError(e.message || 'An unknown error occurred.');
            setIsLoading(false);
            stopPolling();
        }
    }, [prompt, pollOperation, t]);

    useEffect(() => {
        return () => stopPolling();
    }, []);

    return (
        <ToolContainer tool={tool}>
            <div className="space-y-6">
                <div>
                    <label htmlFor="prompt" className="block text-sm font-medium text-slate-700 mb-1">
                        {language === 'en' ? 'Describe the video you want to create' : 'वह वीडियो बताएं जिसे आप बनाना चाहते हैं'}
                    </label>
                    <textarea
                        id="prompt"
                        rows={3}
                        value={prompt}
                        onChange={(e) => setPrompt(e.target.value)}
                        placeholder={language === 'en' ? 'e.g., A neon hologram of a cat driving a sports car at top speed' : 'उदा., एक नीयन होलोग्राम जिसमें एक बिल्ली तेज गति से स्पोर्ट्स कार चला रही है'}
                        className="w-full p-3 border border-slate-300 rounded-lg resize-none focus:ring-2 focus:ring-blue-500 focus:border-blue-500"
                    />
                </div>
                <button
                    onClick={handleGenerateVideo}
                    disabled={!prompt || isLoading}
                    className="w-full flex items-center justify-center bg-blue-600 text-white font-semibold py-2.5 px-4 rounded-lg hover:bg-blue-700 disabled:bg-blue-300 disabled:cursor-not-allowed transition-colors"
                >
                    <WandIcon className="h-5 w-5 mr-2" />
                    {isLoading ? (language === 'en' ? 'Generating...' : 'बना रहा है...') : (language === 'en' ? 'Generate Video' : 'वीडियो बनाएं')}
                </button>
                {error && <p className="text-red-500 text-sm text-center">{error}</p>}
                <div className="mt-6">
                    {isLoading && (
                        <div className="w-full aspect-video bg-slate-100 rounded-lg flex flex-col items-center justify-center text-center p-4">
                            <div className="w-8 h-8 border-4 border-blue-500 border-t-transparent rounded-full animate-spin mb-4"></div>
                            <p className="font-semibold text-slate-700">{loadingMessage}</p>
                        </div>
                    )}
                    {isFetchingVideo && (
                         <div className="w-full aspect-video bg-slate-100 rounded-lg flex flex-col items-center justify-center text-center p-4">
                            <div className="w-8 h-8 border-4 border-blue-500 border-t-transparent rounded-full animate-spin mb-4"></div>
                            <p className="font-semibold text-slate-700">{language === 'en' ? 'Finalizing and loading video...' : 'वीडियो को अंतिम रूप दिया जा रहा है और लोड किया जा रहा है...'}</p>
                        </div>
                    )}
                    {generatedVideoUrl && (
                        <div>
                           <video controls src={generatedVideoUrl} className="w-full rounded-lg border border-slate-200" />
                            <a 
                                href={generatedVideoUrl} 
                                download={`pixelcraft-video-${Date.now()}.mp4`}
                                className="w-full block text-center mt-4 bg-slate-700 text-white font-semibold py-2.5 px-4 rounded-lg hover:bg-slate-800 transition-colors"
                            >
                                {language === 'en' ? 'Download Video' : 'वीडियो डाउनलोड करें'}
                            </a>
                        </div>
                    )}
                </div>
            </div>
        </ToolContainer>
    );
};

const YouTubeThumbnailTool: React.FC<ToolProps> = ({ tool }) => {
    const { language } = useContext(LanguageContext);
    const [topic, setTopic] = useState('');
    const [type, setType] = useState<'long' | 'short'>('long');
    const [generatedImage, setGeneratedImage] = useState<string | null>(null);
    const [isLoading, setIsLoading] = useState<boolean>(false);
    const [error, setError] = useState<string>('');

    const handleGenerate = useCallback(async () => {
        if (!topic) return;
        setIsLoading(true);
        setError('');
        setGeneratedImage(null);

        const aspectRatio = type === 'long' ? '16:9' : '9:16';
        const prompt = `Create a compelling, high-contrast YouTube thumbnail for a video about "${topic}". It should be eye-catching with bold, readable text. Avoid small, cluttered details. This is for a YouTube ${type === 'long' ? 'video' : 'Short'}.`;

        try {
            const imageUrl = await generateImage(prompt, aspectRatio);
            setGeneratedImage(imageUrl);
        } catch (e: any) {
            setError(e.message || 'An unknown error occurred.');
        } finally {
            setIsLoading(false);
        }
    }, [topic, type]);

    return (
        <ToolContainer tool={tool}>
            <div className="space-y-6">
                <div>
                    <label htmlFor="topic" className="block text-sm font-medium text-slate-700 mb-1">
                        {language === 'en' ? 'Video Topic' : 'वीडियो का विषय'}
                    </label>
                    <input
                        type="text"
                        id="topic"
                        value={topic}
                        onChange={(e) => setTopic(e.target.value)}
                        placeholder={language === 'en' ? 'e.g., "10 Tips for Better Sleep"' : 'उदा., "बेहतर नींद के लिए 10 टिप्स"'}
                        className="w-full p-3 border border-slate-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500"
                    />
                </div>
                 <div>
                    <label className="block text-sm font-medium text-slate-700 mb-2">
                        {language === 'en' ? 'Thumbnail Type' : 'थंबनेल प्रकार'}
                    </label>
                    <div className="flex space-x-2">
                        <button onClick={() => setType('long')} className={`px-4 py-2 rounded-lg border text-sm font-semibold transition-colors ${type === 'long' ? 'bg-blue-600 text-white border-blue-600' : 'bg-white text-slate-700 border-slate-300 hover:bg-slate-50'}`}>
                            {language === 'en' ? 'Long Video (16:9)' : 'लंबा वीडियो (16:9)'}
                        </button>
                        <button onClick={() => setType('short')} className={`px-4 py-2 rounded-lg border text-sm font-semibold transition-colors ${type === 'short' ? 'bg-blue-600 text-white border-blue-600' : 'bg-white text-slate-700 border-slate-300 hover:bg-slate-50'}`}>
                            {language === 'en' ? 'Shorts (9:16)' : 'शॉर्ट्स (9:16)'}
                        </button>
                    </div>
                </div>
                 <button
                    onClick={handleGenerate}
                    disabled={!topic || isLoading}
                    className="w-full flex items-center justify-center bg-blue-600 text-white font-semibold py-2.5 px-4 rounded-lg hover:bg-blue-700 disabled:bg-blue-300 disabled:cursor-not-allowed transition-colors"
                >
                    <WandIcon className="h-5 w-5 mr-2" />
                    {isLoading ? (language === 'en' ? 'Generating...' : 'बना रहा है...') : (language === 'en' ? 'Generate Thumbnail' : 'थंबनेल बनाएं')}
                </button>
                 {error && <p className="text-red-500 text-sm text-center">{error}</p>}
                 <div className="mt-6">
                    {isLoading && (
                        <div className={`w-full bg-slate-100 rounded-lg flex items-center justify-center animate-pulse ${type === 'long' ? 'aspect-video' : 'aspect-[9/16] mx-auto max-w-sm'}`}>
                            <ImageIcon className="h-16 w-16 text-slate-300"/>
                        </div>
                    )}
                    {generatedImage && (
                        <div>
                            <img src={generatedImage} alt="Generated thumbnail" className={`w-full rounded-lg border border-slate-200 ${type === 'short' && 'mx-auto max-w-sm'}`} />
                            <a 
                                href={generatedImage} 
                                download={`thumbnail-${Date.now()}.jpg`}
                                className="w-full block text-center mt-4 bg-slate-700 text-white font-semibold py-2.5 px-4 rounded-lg hover:bg-slate-800 transition-colors"
                            >
                                {language === 'en' ? 'Download Thumbnail' : 'थंबनेल डाउनलोड करें'}
                            </a>
                        </div>
                    )}
                </div>
            </div>
        </ToolContainer>
    );
};

const copyToClipboardHelper = (text: string, setCopySuccess: (msg: string) => void) => {
    navigator.clipboard.writeText(text).then(() => {
        setCopySuccess('Copied!');
        setTimeout(() => setCopySuccess(''), 2000);
    }, () => {
        setCopySuccess('Failed!');
        setTimeout(() => setCopySuccess(''), 2000);
    });
};
const YouTubeContentTool: React.FC<ToolProps> = ({ tool }) => {
    const { language } = useContext(LanguageContext);
    const [topic, setTopic] = useState('');
    const [result, setResult] = useState<any>(null);
    const [isLoading, setIsLoading] = useState<boolean>(false);
    const [error, setError] = useState<string>('');
    const [copySuccess, setCopySuccess] = useState('');

    const isScriptGenerator = tool.id === 'youtube-script-generator';
    const contentType = isScriptGenerator ? 'script' : 'description';

    const handleGenerate = useCallback(async () => {
        if (!topic) return;
        setIsLoading(true);
        setError('');
        setResult(null);
        try {
            const content = await generateYouTubeContent(topic, contentType);
            setResult(content);
        } catch (e: any) {
            setError(e.message || 'An unknown error occurred.');
        } finally {
            setIsLoading(false);
        }
    }, [topic, contentType]);

    return (
        <ToolContainer tool={tool}>
            <div className="space-y-6">
                <div>
                    <label htmlFor="topic" className="block text-sm font-medium text-slate-700 mb-1">
                        {language === 'en' ? 'Briefly describe your video topic' : 'अपने वीडियो के विषय का संक्षिप्त विवरण दें'}
                    </label>
                    <textarea
                        id="topic"
                        rows={3}
                        value={topic}
                        onChange={(e) => setTopic(e.target.value)}
                        placeholder={language === 'en' ? 'e.g., "My review of the latest tech gadget"' : 'उदा., "नवीनतम टेक गैजेट की मेरी समीक्षा"'}
                        className="w-full p-3 border border-slate-300 rounded-lg resize-none focus:ring-2 focus:ring-blue-500 focus:border-blue-500"
                    />
                </div>
                <button
                    onClick={handleGenerate}
                    disabled={!topic || isLoading}
                    className="w-full flex items-center justify-center bg-blue-600 text-white font-semibold py-2.5 px-4 rounded-lg hover:bg-blue-700 disabled:bg-blue-300 disabled:cursor-not-allowed transition-colors"
                >
                    <WandIcon className="h-5 w-5 mr-2" />
                    {isLoading ? (language === 'en' ? 'Generating...' : 'बना रहा है...') : (language === 'en' ? 'Generate Content' : 'सामग्री उत्पन्न करें')}
                </button>
                {error && <p className="text-red-500 text-sm text-center">{error}</p>}
                {isLoading && (
                    <div className="space-y-4 animate-pulse">
                        <div className="h-6 bg-slate-200 rounded w-1/4"></div>
                        <div className="h-20 bg-slate-200 rounded"></div>
                        <div className="h-6 bg-slate-200 rounded w-1/4"></div>
                        <div className="h-10 bg-slate-200 rounded"></div>
                    </div>
                )}
                {result && (
                    <div className="space-y-6">
                        {isScriptGenerator ? (
                            <div>
                                <h3 className="text-lg font-semibold text-slate-800 mb-2">{language === 'en' ? 'Generated Script' : 'उत्पन्न स्क्रिप्ट'}</h3>
                                <div className="relative">
                                    <pre className="whitespace-pre-wrap bg-slate-50 p-4 rounded-lg border border-slate-200 font-sans text-sm text-slate-700 h-96 overflow-y-auto">{result}</pre>
                                     <button onClick={() => copyToClipboardHelper(result, setCopySuccess)} className="absolute top-2 right-2 p-2 rounded-md bg-white hover:bg-slate-100 transition">
                                        {copySuccess ? <span className="text-xs text-blue-600">{copySuccess}</span> : <CopyIcon className="h-5 w-5 text-slate-500"/>}
                                    </button>
                                </div>
                            </div>
                        ) : (
                            <>
                                <div>
                                    <h3 className="text-lg font-semibold text-slate-800 mb-2">{language === 'en' ? 'Description' : 'विवरण'}</h3>
                                    <div className="relative">
                                        <p className="bg-slate-50 p-4 rounded-lg border border-slate-200 text-sm text-slate-700">{result.description}</p>
                                        <button onClick={() => copyToClipboardHelper(result.description, setCopySuccess)} className="absolute top-2 right-2 p-2 rounded-md bg-white hover:bg-slate-100 transition">
                                            {copySuccess ? <span className="text-xs text-blue-600">{copySuccess}</span> : <CopyIcon className="h-5 w-5 text-slate-500"/>}
                                        </button>
                                    </div>
                                </div>
                                <div>
                                    <h3 className="text-lg font-semibold text-slate-800 mb-2">{language === 'en' ? 'Tags' : 'टैग'}</h3>
                                    <div className="bg-slate-50 p-4 rounded-lg border border-slate-200 text-sm text-slate-700">
                                        {result.tags.join(', ')}
                                    </div>
                                </div>
                                <div>
                                    <h3 className="text-lg font-semibold text-slate-800 mb-2">{language === 'en' ? 'Hashtags' : 'हैशटैग'}</h3>
                                    <div className="bg-slate-50 p-4 rounded-lg border border-slate-200 text-sm text-slate-700">
                                        {result.hashtags.map((h: string) => `#${h}`).join(' ')}
                                    </div>
                                </div>
                            </>
                        )}
                    </div>
                )}
            </div>
        </ToolContainer>
    );
};


// --- CONSTANTS ---
const TOOLS: Tool[] = [
    { id: 'image-to-text', name: { en: 'Image to Text', hi: 'छवि से टेक्स्ट' }, description: { en: 'Extract text from any image.', hi: 'किसी भी छवि से टेक्स्ट निकालें।' }, category: 'image', icon: TextIcon, isImplemented: true, },
    { id: 'ai-image-generator', name: { en: 'AI Image Generator', hi: 'एआई छवि जेनरेटर' }, description: { en: 'Create images from text descriptions.', hi: 'टेक्स्ट विवरण से छवियां बनाएं।' }, category: 'image', icon: WandIcon, isImplemented: true, },
    { id: 'image-compressor', name: { en: 'Image Compressor', hi: 'छवि कंप्रेसर' }, description: { en: 'Reduce image file size.', hi: 'छवि फ़ाइल का आकार कम करें।' }, category: 'image', icon: CompressIcon, isImplemented: false, },
    { id: 'image-editor', name: { en: 'Image Editor', hi: 'छवि संपादक' }, description: { en: 'Edit and enhance your images.', hi: 'अपनी छवियों को संपादित और बेहतर बनाएं।' }, category: 'image', icon: ImageIcon, isImplemented: false, },
    { id: 'background-remover', name: { en: 'Background Remover', hi: 'पृष्ठभूमि हटानेवाला' }, description: { en: 'Remove image backgrounds.', hi: 'छवि पृष्ठभूमि हटाएं।' }, category: 'image', icon: ScissorsIcon, isImplemented: false, },
    { id: 'ai-video-generator', name: { en: 'AI Video Generator', hi: 'एआई वीडियो जेनरेटर' }, description: { en: 'Generate videos from text prompts.', hi: 'टेक्स्ट प्रॉम्प्ट से वीडियो जेनरेट करें।' }, category: 'video', icon: VideoIcon, isImplemented: true, },
    { id: 'video-editor', name: { en: 'Video Editor', hi: 'वीडियो संपादक' }, description: { en: 'A simple yet powerful video editor.', hi: 'एक सरल लेकिन शक्तिशाली वीडियो संपादक।' }, category: 'video', icon: ScissorsIcon, isImplemented: false, },
    { id: 'youtube-thumbnail', name: { en: 'YouTube Thumbnail Generator', hi: 'यूट्यूब थंबनेल जेनरेटर' }, description: { en: 'Create stunning thumbnails for videos.', hi: 'वीडियो के लिए शानदार थंबनेल बनाएं।' }, category: 'youtube', icon: YoutubeIcon, isImplemented: true, },
    { id: 'youtube-description-tags', name: { en: 'YouTube Description & Tags', hi: 'यूट्यूब विवरण और टैग' }, description: { en: 'Generate video descriptions and tags.', hi: 'वीडियो विवरण और टैग जेनरेट करें।' }, category: 'youtube', icon: TagIcon, isImplemented: true, },
    { id: 'youtube-script-generator', name: { en: 'YouTube Script Generator', hi: 'यूट्यूब स्क्रिप्ट जेनरेटर' }, description: { en: 'Generate scripts for your videos.', hi: 'अपने वीडियो के लिए स्क्रिप्ट जेनरेट करें।' }, category: 'youtube', icon: ScriptIcon, isImplemented: true, },
    { id: 'word-to-voice', name: { en: 'Word to Voice', hi: 'शब्द से आवाज' }, description: { en: 'Convert text into natural speech.', hi: 'पाठ को प्राकृतिक भाषण में बदलें।' }, category: 'audio', icon: MicIcon, isImplemented: false, },
    { id: 'background-music', name: { en: 'Background Music', hi: 'पार्श्व संगीत' }, description: { en: 'Generate background music for videos.', hi: 'वीडियो के लिए पृष्ठभूमि संगीत उत्पन्न करें।' }, category: 'audio', icon: MusicIcon, isImplemented: false, },
    { id: 'qr-code-design', name: { en: 'QR Code Designer', hi: 'क्यूआर कोड डिजाइनर' }, description: { en: 'Create stylish custom QR codes.', hi: 'स्टाइलिश कस्टम क्यूआर कोड बनाएं।' }, category: 'qr', icon: QrCodeIcon, isImplemented: false, },
    { id: 'media-to-qr', name: { en: 'Media to QR Code', hi: 'मीडिया से क्यूआर कोड' }, description: { en: 'Link images and videos to a QR code.', hi: 'छवियों और वीडियो को क्यूआर कोड से लिंक करें।' }, category: 'qr', icon: QrCodeIcon, isImplemented: false, },
    { id: 'qr-code-generator', name: { en: 'QR Code Generator', hi: 'क्यूआर कोड जेनरेटर' }, description: { en: 'Generate simple QR codes.', hi: 'सरल क्यूआर कोड जेनरेट करें।' }, category: 'qr', icon: QrCodeIcon, isImplemented: false, },
    { id: 'pdf-editor', name: { en: 'PDF Editor', hi: 'पीडीएफ संपादक' }, description: { en: 'Edit text and images in PDF files.', hi: 'पीडीएफ फाइलों में टेक्स्ट और इमेज संपादित करें।' }, category: 'document', icon: FileTextIcon, isImplemented: false, },
];
const CONTACT_EMAIL = 'tinkuvishal40@gmail.com';
const CONTACT_PHONE = '7973831205';
const i18n = {
    en: { logo: 'PixelCraft Suite', searchPlaceholder: 'Search for a tool...', contactUs: 'Contact Us', language: 'Language', english: 'English', hindi: 'Hindi', dashboard: 'Dashboard', allTools: 'All Tools', imageTools: 'Image Tools', videoTools: 'Video Tools', youtubeTools: 'YouTube Tools', audioTools: 'Audio Tools', qrTools: 'QR Code Tools', documentTools: 'Document Tools', toolComingSoon: 'This tool is coming soon!', toolComingSoonDesc: 'We are working hard to bring this feature to you. Please check back later.', goBack: 'Go Back to Dashboard', },
    hi: { logo: 'पिक्सेलक्राफ्ट सुइट', searchPlaceholder: 'एक टूल खोजें...', contactUs: 'हमसे संपर्क करें', language: 'भाषा', english: 'अंग्रेजी', hindi: 'हिंदी', dashboard: 'डैशबोर्ड', allTools: 'सभी उपकरण', imageTools: 'छवि उपकरण', videoTools: 'वीडियो उपकरण', youtubeTools: 'यूट्यूब उपकरण', audioTools: 'ऑडियो उपकरण', qrTools: 'क्यूआर कोड उपकरण', documentTools: 'दस्तावेज़ उपकरण', toolComingSoon: 'यह टूल जल्द ही आ रहा है!', toolComingSoonDesc: 'हम इस सुविधा को आप तक लाने के लिए कड़ी मेहनत कर रहे हैं। कृपया बाद में फिर देखें।', goBack: 'डैशबोर्ड पर वापस जाएं', }
};
const CATEGORIES = {
    image: { name: { en: 'Image Tools', hi: 'छवि उपकरण' } },
    video: { name: { en: 'Video Tools', hi: 'वीडियो उपकरण' } },
    youtube: { name: { en: 'YouTube Tools', hi: 'यूट्यूब उपकरण' } },
    audio: { name: { en: 'Audio Tools', hi: 'ऑडियो उपकरण' } },
    qr: { name: { en: 'QR Code Tools', hi: 'क्यूआर कोड उपकरण' } },
    document: { name: { en: 'Document Tools', hi: 'दस्तावेज़ उपकरण' } },
};

// --- LAYOUT COMPONENTS ---
const Header: React.FC = () => {
    const { language, setLanguage } = useContext(LanguageContext);
    const t = i18n[language];
    const [isContactOpen, setIsContactOpen] = useState(false);
    const [isLangOpen, setIsLangOpen] = useState(false);
    const contactRef = useRef<HTMLDivElement>(null);
    const langRef = useRef<HTMLDivElement>(null);

    useEffect(() => {
        const handleClickOutside = (event: MouseEvent) => {
            if (contactRef.current && !contactRef.current.contains(event.target as Node)) setIsContactOpen(false);
            if (langRef.current && !langRef.current.contains(event.target as Node)) setIsLangOpen(false);
        };
        document.addEventListener("mousedown", handleClickOutside);
        return () => document.removeEventListener("mousedown", handleClickOutside);
    }, []);

    return (
        <header className="flex-shrink-0 bg-white h-16 border-b border-slate-200 flex items-center justify-between px-4 sm:px-6">
            <div className="flex items-center space-x-3">
                <LogoIcon className="h-8 w-8 text-blue-600" />
                <h1 className="text-xl font-bold text-slate-800 hidden sm:block">{t.logo}</h1>
            </div>
            <div className="flex items-center space-x-4">
                <div className="relative" ref={contactRef}>
                    <button onClick={() => setIsContactOpen(!isContactOpen)} className="text-sm font-semibold text-slate-600 hover:text-blue-600">{t.contactUs}</button>
                    {isContactOpen && (
                        <div className="absolute right-0 mt-2 w-64 bg-white border rounded-lg shadow-lg z-10 p-4">
                           <div className="flex items-center space-x-3 mb-3"><MailIcon className="h-5 w-5 text-slate-500" /><a href={`mailto:${CONTACT_EMAIL}`} className="text-sm text-slate-700 hover:text-blue-600">{CONTACT_EMAIL}</a></div>
                           <div className="flex items-center space-x-3"><PhoneIcon className="h-5 w-5 text-slate-500" /><a href={`tel:${CONTACT_PHONE}`} className="text-sm text-slate-700 hover:text-blue-600">{CONTACT_PHONE}</a></div>
                        </div>
                    )}
                </div>
                <div className="relative" ref={langRef}>
                     <button onClick={() => setIsLangOpen(!isLangOpen)} className="text-sm font-semibold text-slate-600 hover:text-blue-600">{t.language}</button>
                    {isLangOpen && (
                        <div className="absolute right-0 mt-2 w-32 bg-white border rounded-lg shadow-lg z-10">
                            <button onClick={() => { setLanguage('en'); setIsLangOpen(false); }} className={`w-full text-left px-4 py-2 text-sm ${language === 'en' ? 'bg-blue-50 text-blue-600' : 'text-slate-700 hover:bg-slate-100'}`}>{t.english}</button>
                            <button onClick={() => { setLanguage('hi'); setIsLangOpen(false); }} className={`w-full text-left px-4 py-2 text-sm ${language === 'hi' ? 'bg-blue-50 text-blue-600' : 'text-slate-700 hover:bg-slate-100'}`}>{t.hindi}</button>
                        </div>
                    )}
                </div>
            </div>
        </header>
    );
};

interface SidebarProps {
    activeToolId: ToolId | 'dashboard';
    setActiveToolId: (id: ToolId | 'dashboard') => void;
}
const Sidebar: React.FC<SidebarProps> = ({ activeToolId, setActiveToolId }) => {
    const { language } = useContext(LanguageContext);
    const t = i18n[language];
    const categoriesOrder: (keyof typeof CATEGORIES)[] = ['image', 'video', 'youtube', 'audio', 'qr', 'document'];

    return (
        <nav className="w-64 bg-white border-r border-slate-200 p-4 flex-col h-full overflow-y-auto hidden md:flex">
            <button onClick={() => setActiveToolId('dashboard')} className={`flex items-center space-x-3 px-3 py-2.5 rounded-lg text-sm font-semibold mb-4 ${activeToolId === 'dashboard' ? 'bg-blue-50 text-blue-600' : 'text-slate-600 hover:bg-slate-100'}`}>
                <HomeIcon className="h-5 w-5" />
                <span>{t.dashboard}</span>
            </button>
            <div className="space-y-4">
                {categoriesOrder.map(catKey => (
                    <div key={catKey}>
                        <h3 className="px-3 text-xs font-semibold text-slate-400 uppercase tracking-wider mb-2">{CATEGORIES[catKey].name[language]}</h3>
                        <div className="space-y-1">
                            {TOOLS.filter(tool => tool.category === catKey).map(tool => (
                                <button key={tool.id} onClick={() => setActiveToolId(tool.id)} className={`w-full flex items-center space-x-3 px-3 py-2 rounded-lg text-sm text-left ${activeToolId === tool.id ? 'bg-blue-50 text-blue-600 font-semibold' : 'text-slate-600 hover:bg-slate-100'}`}>
                                    <tool.icon className="h-5 w-5 flex-shrink-0" />
                                    <span>{tool.name[language]}</span>
                                </button>
                            ))}
                        </div>
                    </div>
                ))}
            </div>
        </nav>
    );
};

interface ToolCardProps { tool: Tool; onClick: () => void; }
const ToolCard: React.FC<ToolCardProps> = ({ tool, onClick }) => {
    const { language } = useContext(LanguageContext);
    return (
        <button onClick={onClick} className="bg-white p-5 rounded-xl border border-slate-200 hover:border-blue-500 hover:shadow-lg transition-all text-left flex flex-col items-start">
            <div className="bg-blue-100 text-blue-600 rounded-lg p-3 mb-4"><tool.icon className="h-6 w-6" /></div>
            <h3 className="font-semibold text-slate-800 mb-1">{tool.name[language]}</h3>
            <p className="text-sm text-slate-500 flex-grow">{tool.description[language]}</p>
        </button>
    );
};

interface DashboardProps { onSelectTool: (id: ToolId) => void; }
const Dashboard: React.FC<DashboardProps> = ({ onSelectTool }) => {
    const { language } = useContext(LanguageContext);
    const t = i18n[language];
    const [searchTerm, setSearchTerm] = useState('');
    const categoriesOrder: (keyof typeof CATEGORIES)[] = ['image', 'video', 'youtube', 'audio', 'qr', 'document'];

    const filteredTools = useMemo(() => {
        if (!searchTerm) return TOOLS;
        return TOOLS.filter(tool =>
            tool.name[language].toLowerCase().includes(searchTerm.toLowerCase()) ||
            tool.description[language].toLowerCase().includes(searchTerm.toLowerCase())
        );
    }, [searchTerm, language]);

    return (
        <div className="max-w-7xl mx-auto">
            <div className="text-center mb-8">
                <h1 className="text-4xl font-bold text-slate-800 mb-2">{t.logo}</h1>
                <p className="text-lg text-slate-500">Your all-in-one AI creative toolkit.</p>
            </div>
            <div className="relative mb-10 max-w-xl mx-auto">
                <SearchIcon className="absolute left-4 top-1/2 -translate-y-1/2 h-5 w-5 text-slate-400" />
                <input type="text" placeholder={t.searchPlaceholder} value={searchTerm} onChange={(e) => setSearchTerm(e.target.value)} className="w-full pl-12 pr-4 py-3 border border-slate-300 rounded-full focus:ring-2 focus:ring-blue-500 outline-none" />
            </div>
            <div className="space-y-10">
                {searchTerm ? (
                     <div className="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-6">
                        {filteredTools.map(tool => <ToolCard key={tool.id} tool={tool} onClick={() => onSelectTool(tool.id)} />)}
                    </div>
                ) : (
                    categoriesOrder.map(catKey => {
                        const toolsInCategory = TOOLS.filter(tool => tool.category === catKey);
                        if (toolsInCategory.length === 0) return null;
                        return (
                            <div key={catKey}>
                                <h2 className="text-2xl font-bold text-slate-800 mb-4">{CATEGORIES[catKey].name[language]}</h2>
                                <div className="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-6">
                                    {toolsInCategory.map(tool => <ToolCard key={tool.id} tool={tool} onClick={() => onSelectTool(tool.id)} />)}
                                </div>
                            </div>
                        )
                    })
                )}
                 {filteredTools.length === 0 && searchTerm && <p className="text-center text-slate-500 mt-8">No tools found for "{searchTerm}".</p>}
            </div>
        </div>
    );
};


// --- MAIN APP COMPONENT ---
const App: React.FC = () => {
    const [activeToolId, setActiveToolId] = useState<ToolId | 'dashboard'>('dashboard');
    const [language, setLanguage] = useState<Language>('en');
    const languageContextValue = useMemo(() => ({ language, setLanguage }), [language]);

    const renderActiveTool = () => {
        const tool = TOOLS.find(t => t.id === activeToolId);
        if (!tool || activeToolId === 'dashboard') {
            return <Dashboard onSelectTool={setActiveToolId} />;
        }
        if (!tool.isImplemented) {
            return <PlaceholderTool tool={tool} />;
        }
        
        switch (activeToolId) {
            case 'image-to-text': return <ImageToTextTool tool={tool} />;
            case 'ai-image-generator': return <ImageGeneratorTool tool={tool} />;
            case 'ai-video-generator': return <VideoGeneratorTool tool={tool} />;
            case 'youtube-thumbnail': return <YouTubeThumbnailTool tool={tool} />;
            case 'youtube-description-tags':
            case 'youtube-script-generator': return <YouTubeContentTool tool={tool} />;
            default: return <PlaceholderTool tool={tool} />;
        }
    };

    return (
        <LanguageContext.Provider value={languageContextValue}>
            <div className="flex h-screen bg-white font-sans">
                <Sidebar activeToolId={activeToolId} setActiveToolId={setActiveToolId} />
                <div className="flex flex-col flex-1">
                    <Header />
                   <main className="flex-1 overflow-y-auto bg-slate-50 p-4 sm:p-6 md:p-8">
                        {renderActiveTool()}
                     </main>
                </div>
            </div>
        </LanguageContext.Provider>
    );
};


// --- RENDER APP ---
const rootElement = document.getElementById('root');
if (!rootElement) throw new Error("Could not find root element to mount to");

const root = ReactDOM.createRoot(rootElement);
root.render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);
